# DTrace Program Structure 

## 1. **Basic Program Structure**

### General Form
```d
probe_description
/ predicate /
{
    action statements
}
```

### Components
- **Probe descriptions** - Required (which probes to enable)
- **Predicate** - Optional (conditional filter)
- **Actions** - Optional (what to execute)

### Example
```d
syscall::read:entry
/ pid == 12345 /
{
    trace(timestamp);
}
```

---

## 2. **Probe Descriptions**

### Standard Format
```
provider:module:function:name
```

### Field Matching Rules
- Fields are interpreted **right-to-left** if omitted
- `foo:bar` = any provider/module, function=foo, name=bar
- **Best practice:** Always specify all 4 fields with delimiters

### Examples
```d
syscall:::           /* All syscall provider probes */
syscall::read:entry  /* Specific probe */
:::entry             /* All entry probes (any provider) */
```

---

## 3. **Pattern Matching**

### Wildcards (Shell-style Globbing)

| Character | Description | Example |
|-----------|-------------|---------|
| `*` | Match any string (including empty) | `syscall::*read*:entry` |
| `?` | Match single character | `syscall::rea?:entry` |
| `[...]` | Match any enclosed character | `syscall::[rw]ead:entry` |
| `[!...]` | Match any NOT enclosed | `syscall::[!r]*:entry` |
| `\` | Escape special character | `syscall::\*foo:entry` |

### Pattern Examples
```d
/* All syscalls starting with "lwp" or "sock" */
syscall::*lwp*:entry, syscall::*sock*:entry
{
    trace(timestamp);
}

/* List probes with dtrace command */
dtrace -l -f kmem_*  /* All functions starting with kmem_ */
```

---

## 4. **Multiple Probe Descriptions**

### Comma-Separated Lists
Apply same predicate/actions to multiple probes:

```d
probe1, probe2, probe3
/ predicate /
{
    actions
}
```

### Example
```d
syscall::read:entry, syscall::write:entry
/ pid == 1234 /
{
    printf("I/O syscall by process %d\n", pid);
}
```

---

## 5. **Predicates**

### Purpose
- Filter probe firings conditionally
- Enclosed in `/ /` slashes
- Must evaluate to integer/pointer (0=false, non-zero=true)

### Examples
```d
/* Only trace specific PID */
syscall::read:entry
/ pid == 12345 /
{ trace(arg0); }

/* Only trace if variable is set */
syscall::read:entry
/ self->traced /
{ trace(timestamp); }

/* Multiple conditions */
syscall::write:entry
/ pid == 1234 && arg0 > 100 /
{ printf("Large write\n"); }

/* No predicate - always execute */
syscall::read:entry
{ trace(timestamp); }
```

---

## 6. **Actions**

### Characteristics
- Statements enclosed in `{ }`
- Separated by semicolons `;`
- Can be empty (just note probe fired)

### Examples
```d
/* Empty action - just count firings */
syscall::read:entry
{ }

/* Single action */
syscall::read:entry
{ trace(arg0); }

/* Multiple actions */
syscall::read:entry
{
    printf("Read called\n");
    trace(timestamp);
    self->count++;
}
```

---

## 7. **Declarations**

### Placement Rules
- **Outside probe clauses only**
- Cannot be inside `{ }` braces
- Cannot be between probe clause elements

### What Can Be Declared
```d
/* Variable declarations */
int x;
self int y;
string name;

/* Type definitions */
typedef struct mydata {
    int value;
    string name;
} mydata_t;

/* External C symbols */
extern int `kernel_variable`;
```

---

## 8. **Program Layout**

### Correct Structure
```d
/* Declarations first */
int global_counter;
self int thread_local;

/* Probe clauses next */
syscall::read:entry
{
    global_counter++;
}

syscall::write:entry
/ global_counter > 10 /
{
    trace(timestamp);
}
```

### Invalid Structure ❌
```d
syscall::read:entry
{
    int x;  /* ❌ Cannot declare inside actions */
}

int counter;  /* ✅ OK */
syscall::read:entry
int another;  /* ❌ Cannot declare between elements */
{
    trace(counter);
}
```

---

## 9. **C Preprocessor Integration**

### Usage
Enable with `-C` flag:
```bash
dtrace -C -s program.d
```

### What It Does
- Executes `cpp` before D compilation
- Enables `#include`, `#define`, macros
- Access to C header files and types

### Example
```d
#define TRACE_PID 12345

/* Include system header */
#include <sys/types.h>

syscall::read:entry
/ pid == TRACE_PID /
{
    trace(timestamp);
}
```

### Restrictions
- Only include files with **valid D declarations**
- Cannot include C function source code
- Typical C headers (type declarations) work fine

---

## 10. **Pragmas**

### Purpose
Compiler directives starting with `#`

### Example
```d
#pragma D option quiet
#pragma D option switchrate=10hz

BEGIN
{
    printf("Starting trace...\n");
}
```

---

## Complete Example

```d
/* ========================================
 * Complete D Program Structure Example
 * ======================================== */

/* Declarations */
int read_count;
self int in_read;

/* First probe clause */
syscall::read:entry
/ execname == "myapp" /
{
    self->in_read = 1;
    self->start = timestamp;
    read_count++;
}

/* Second probe clause */
syscall::read:return
/ self->in_read /
{
    printf("%s: read took %d ns\n", 
           execname, timestamp - self->start);
    self->in_read = 0;
    self->start = 0;
}

/* Multiple probes, one clause */
syscall::write:entry, syscall::open:entry
{
    printf("%s called by %d\n", probename, pid);
}

/* END clause */
END
{
    printf("Total reads: %d\n", read_count);
}
```
