# DTrace Variables - Quick Reference Guide
DTrace provides variables for tracking data in tracing programs. Variables are automatically created on first assignment and initially filled with zeros.

---

## 1. **Scalar Variables** (Global)

### Characteristics
- Represent individual fixed-size data (integers, pointers, strings)
- **Global scope** - visible everywhere in your program
- Created automatically on first assignment

### Example
```d
BEGIN {
    x = 123;  // Creates global int variable x
}
```

### Optional Declaration
```d
int x;  /* explicit declaration */

BEGIN {
    x = 123;
}
```

---

## 2. **Associative Arrays**

### Characteristics
- Collections indexed by **tuples** (key combinations)
- Dynamic size (no predefined limit)
- Any scalar type can be key or value
- Unassigned elements return zero
- **Assign zero to free memory** when done

### Example
```d
a[123, "hello"] = 456;  // Key: [int, string], Value: int
```

### Declaration (optional)
```d
int x[unsigned long long, char];

BEGIN {
    x[123ull, 'a'] = 456;
}
```

---

## 3. **Thread-Local Variables** (`self->`)

### Characteristics
- Separate storage **per thread**
- Use `self->` operator
- Automatically indexed by thread identity
- Perfect for tracking per-thread state

### Example
```d
syscall::read:entry {
    self->t = timestamp;  // Each thread gets own copy
}

syscall::read:return {
    printf("Time: %d ns\n", timestamp - self->t);
    self->t = 0;  // Free memory when done
}
```

### Declaration (optional)
```d
self int x;

syscall::read:entry {
    self->x = 123;
}
```

---

## 4. **Clause-Local Variables** (`this->`)

### Characteristics
- Storage reused **per clause execution**
- Use `this->` operator
- Persist across multiple clauses for same probe
- **Fastest access** (no associative array lookup)
- Values undefined at start - must initialize first!

### Example
```d
BEGIN {
    this->secs = timestamp / 1000000000;
}
```

### Multi-Clause Persistence
```d
tick-1sec {
    this->foo = 10;
    printf("foo is %d\n", this->foo++);
}

tick-1sec {
    printf("foo is still %d\n", this->foo++);  // Persists!
}
```

---

## 5. **Built-in Variables**

### Common Built-ins
| Variable | Description |
|----------|-------------|
| `arg0`...`arg9` | First 10 probe arguments (raw 64-bit) |
| `args[]` | Typed probe arguments |
| `pid` | Process ID |
| `tid` | Thread ID |
| `execname` | Program name |
| `timestamp` | Nanosecond counter (relative) |
| `walltimestamp` | Nanoseconds since Unix epoch |
| `probename` | Current probe name |
| `probefunc` | Current probe function |
| `cpu` | Current CPU ID |
| `errno` | Last system call error |

---

## 6. **External Variables** (`` `variable ``)

### Characteristics
- Access kernel/OS variables using backtick (`` ` ``)
- Read-only (cannot modify)
- Based on kernel source code types
- **Unstable interface** - may break with OS updates

### Example
```d
BEGIN {
    printf("kmem_flags = %d\n", `kmem_flags);
}
```

### Module-Specific Access
```d
foo`_fini  // Access _fini from module "foo"
```

---

## Quick Comparison Table

| Type | Syntax | Scope | Use Case |
|------|--------|-------|----------|
| **Global** | `x = 1` | All clauses | Shared state |
| **Associative Array** | `a[key] = val` | All clauses | Key-value storage |
| **Thread-Local** | `self->x = 1` | Per thread | Track per-thread data |
| **Clause-Local** | `this->x = 1` | Per clause | Temporary calculations |
| **Built-in** | `pid`, `timestamp` | - | System information |
| **External** | `` `kmem_flags` `` | Kernel | OS internals (read-only) |

