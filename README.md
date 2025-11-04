# DTrace 
<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/aab15842-23c4-4cf2-bd96-588f36b72c29" />

## 📚 Modules

### 1. [Introduction](https://github.com/khangpt2k6/Dtrace/blob/main/README.md)
Getting started with DTrace basics and probe syntax.

### 2. [Types, Operators & Expressions](https://github.com/khangpt2k6/Dtrace/blob/main/type_operator_expression.md)
Data types, operators, and expression evaluation in D language.

### 3. [Variables](https://github.com/khangpt2k6/Dtrace/blob/main/variable.md)
Variable scopes, declarations, and state management.

### 4. [Program Structure](https://github.com/khangpt2k6/Dtrace/blob/main/structure.md)
How D programs are organized, including modules, functions, and control flow.

### 5. [Pointers & Arrays](https://github.com/khangpt2k6/Dtrace/blob/main/pointers_array.md)
Pointer fundamentals, arrays, pointer arithmetic, and address spaces.

### 6. [Strings](https://github.com/khangpt2k6/Dtrace/blob/main/string.md)
String manipulation

## Overview

**DTrace** stands for **Dynamic Tracing**.

- It is a performance-analysis and troubleshooting framework / tool suite built into (or available for) certain operating systems (e.g., Solaris family, FreeBSD, macOS) by default.

> **Note:**  
> What makes it special is that it can instrument **every layer of the software stack** — from user applications and system libraries all the way down to the kernel and device drivers.

- It provides its own scripting or command-language (called **“D”**) which is similar to **C** and **awk**.  

- This allows flexible filtering and summarization of trace data **in-kernel** before passing to user space, helping keep overhead low enough for production usage.

### Example

Tracing which processes are executed when you run “man ls”, along with nanosecond timestamps:

```bash
# dtrace -n 'proc:::exec-success { printf("%d %s", timestamp, curpsinfo->pr_psargs); }'
dtrace: description 'proc:::exec-success ' matched 1 probe
CPU     ID                  FUNCTION:NAME
  1    797       exec_common:exec-success 21935388676181394 man ls
  0    797       exec_common:exec-success 21935388840101743 sh -c cd /usr/share/man; tbl /usr/share/man/man1/ls.1 |neqn /usr/share/lib/pub/
  1    797       exec_common:exec-success 21935388858652639 col -x
  0    797       exec_common:exec-success 21935388863714971 neqn /usr/share/lib/pub/eqnchar -
  0    797       exec_common:exec-success 21935388867119787 tbl /usr/share/man/man1/ls.1
  1    797       exec_common:exec-success 21935388881310626 nroff -u0 -Tlp -man -
  0    797       exec_common:exec-success 21935388922364648 sh -c trap '' 1 15; /usr/bin/mv -f /tmp/mpDKaiGn /usr/share/man/cat1/ls.1 2> /d
  1    797       exec_common:exec-success 21935388930987671 /usr/bin/mv -f /tmp/mpDKaiGn /usr/share/man/cat1/ls.1
  1    797       exec_common:exec-success 21935388948485045 sh -c less -siM /tmp/mpDKaiGn
  0    797       exec_common:exec-success 21935388971039204 less -siM /tmp/mpDKaiGn
```

### How to read this output

- **CPU**: the logical CPU core where the probe fired (e.g., `1` = core 1). This is not a percentage by itself.
- **ID**: the DTrace probe ID (internal identifier for the enabled probe). It is not a PID.
- **FUNCTION:NAME**: the provider function and probe name that fired (here `exec_common:exec-success`).
- **Payload**: the rest is produced by your `printf` — first the nanosecond `timestamp`, then the process arguments (e.g., `man ls`).

> Note: To compute CPU utilization percentage, you need to aggregate time or samples over an interval (see CPU monitoring below). The CPU column alone only shows where an event occurred.

---

## Common Use Cases

### 1. Process Monitoring

**Trace new process execution:**
```bash
dtrace -n 'proc:::exec-success { trace(curpsinfo->pr_psargs); }'
```

**Monitor file access:**
```bash
dtrace -n 'syscall::open*:entry { printf("%s %s",execname,copyinstr(arg0)); }'
```

### 2. Performance Analysis

**System call statistics by program:**
```bash
dtrace -n 'syscall:::entry { @num[execname] = count(); }'
```

**I/O throughput measurement:**
```bash
dtrace -n 'sysinfo:::readch { @bytes[execname] = sum(arg0); }'
dtrace -n 'sysinfo:::writech { @bytes[execname] = sum(arg0); }'
```

**CPU profiling:**
```bash
dtrace -n 'profile-99 /pid == 189 && arg1/ { @[ustack()] = count(); }'
```

**CPU utilization (sampling-based):** Low-overhead approximation of on-CPU share.
```bash
sudo dtrace -n '
profile-99
{
  @samples[execname, pid] = count();
}
tick-1s
{
  normalize(@samples, 99);                /* samples/sec per core */
  printa("%-16s pid=%-6d cpu%≈%d\n", @samples);
  clear(@samples);
}'
```
- Interpretation: per-process cpu% ≈ samples_per_sec. On multi-core systems, a busy process can exceed 100% by using multiple cores.

**CPU utilization (scheduler-based, precise on-CPU time):**
```bash
sudo dtrace -n '
sched:::on-cpu
/ pid != 0 /
{
  self->start = timestamp;
}
sched:::off-cpu
/ self->start /
{
  @oncpu_ns[execname, pid] = sum(timestamp - self->start);
  self->start = 0;
}
tick-1s
{
  /* Convert to ms for readability */
  normalize(@oncpu_ns, 1000000);
  printa("%-16s pid=%-6d oncpu_ms=%-8d\n", @oncpu_ns);
  clear(@oncpu_ns);
}'
```
- Per-process CPU% (per core) in the interval: `cpu% = oncpu_ms / 1000 × 100`.
- System average across `N` cores: `sum(oncpu_ms) / (1000 × N) × 100`.

### 3. Disk I/O Analysis

**Real-time disk activity monitoring:**
```bash
dtrace -n 'io:::start { printf("%d %s %d",pid,execname,args[0]->b_bcount); }'
```

This command displays:
- Process ID performing I/O operations
- Process name
- Transfer size in bytes
- Block address locations

### 4. Network Activity

**Per-process throughput (bytes/sec) using IP providers:**
```bash
sudo dtrace -n '
ip:::send
/ pid != 0 /
{
  @tx[execname, pid] = sum(args[2]->ip_plength);
}
ip:::receive
/ pid != 0 /
{
  @rx[execname, pid] = sum(args[2]->ip_plength);
}
tick-1s
{
  printa("TX %-16s pid=%-6d %10@d B/s\n", @tx);
  printa("RX %-16s pid=%-6d %10@d B/s\n", @rx);
  clear(@tx); clear(@rx);
}'
```

- **Link utilization %** for a link with capacity `S` bytes/sec: `(TX_Bps + RX_Bps) / S × 100`.
- Replace or complement with `tcp:::send`/`tcp:::receive` if you want TCP-only.

### 5. Memory Analysis

**Page fault tracking:**
```bash
dtrace -n 'vminfo:::as_fault { @mem[execname] = sum(arg0); }'
```

**Page-in monitoring:**
```bash
dtrace -n 'vminfo:::pgpgin { @pg[execname] = sum(arg0); }'
```

**Per-process resident memory (RSS) in KB:**
```bash
sudo dtrace -n '
tick-1s
/ pid != 0 /
{
  @rss_kb[execname, pid] = quantize(curpsinfo->pr_rssize * (pagesize / 1024));
}
tick-5s
{
  printa("%-16s pid=%-6d RSS(KB)\n%@d", @rss_kb);
  clear(@rss_kb);
}'
```
- **Memory %** vs total RAM: `memory% = rss_bytes / total_ram_bytes × 100`.
- Use allocation probes (e.g., `syscall::mmap*:entry/return`, `syscall::munmap*:entry`) to study growth and leaks.

---

## Key Advantages

| Advantage | Features |
|-----------|----------|
| **Safe for Production Environments** | Minimal system overhead<br>Built-in safety checks<br>No system downtime required |
| **Comprehensive Visibility** | Full-stack instrumentation<br>Kernel and user-space coverage<br>Hardware-level insights |
| **Real-Time Monitoring** | Live event observation<br>Immediate data collection<br>No post-processing delays |
| **Flexible Implementation** | Simple one-liners for quick analysis<br>Complex scripts for detailed investigation<br>Customizable aggregation functions |
| **Efficient Data Processing** | In-kernel filtering and summarization<br>Reduces data transfer overhead<br>Scales to high-frequency events |
| **Zero Preparation** | Works with existing binaries<br>No source code modification<br>No recompilation necessary |


## Resources

- [DTrace Official Site](https://dtrace.org/about/)
- [Brendan Gregg’s DTrace Guide](https://www.brendangregg.com/dtrace.html)


