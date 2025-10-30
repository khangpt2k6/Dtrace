# D Language — Types, Operators, and Expressions Summary

## 2.1 Identifier Names & Keywords

**Identifiers:**  
Composed of letters, digits, and underscores (`_`); must start with a letter or `_`.  
Names starting with `_` are reserved.  

**Naming conventions:**  
- `MixedCase` → variables  
- `UPPERCASE` → constants  

**Keywords:**  
- Superset of ANSI-C keywords  
- `*` → reserved for future use  
- `+` → D-specific (e.g., `self`, `translator`, `xlate`)  
- No control-flow constructs (`if`, `for`, etc.) allowed yet  

---

## 2.2 Data Types and Sizes

Arithmetic is only allowed on **integers** (not floating-point types).  

**Data model:** 32-bit or 64-bit (check via `isainfo -b`).  

### Integer Types

| Type       | 32-bit | 64-bit |
|-------------|---------|---------|
| `char`      | 1 byte  | 1 byte  |
| `short`     | 2 bytes | 2 bytes |
| `int`       | 4 bytes | 4 bytes |
| `long`      | 4 bytes | 8 bytes |
| `long long` | 8 bytes | 8 bytes |

- Default: signed unless specified as unsigned  
- Type aliases: `int8_t`, `uint32_t`, `intptr_t`, etc.  

### Floating-Point Types

| Type          | Size     |
|----------------|----------|
| `float`        | 4 bytes  |
| `double`       | 8 bytes  |
| `long double`  | 16 bytes |

- No floating-point arithmetic — only tracing and formatting  
- `string` → special type for ASCII strings  

---

## 2.3 Constants

**Integers:**  
- Decimal (`123`), Octal (`0123`), Hex (`0x123`)  

**Suffixes:**  
- `u/U`: unsigned  
- `l/L`: long  
- `ll/LL`: long long  
- `ul/UL`, `ull/ULL`: unsigned variants  

**Floating-point constants:**  
- Decimal or exponent form  
- Default type: `double`  
- Suffixes: `f/F` (float), `l/L` (long double)  

**Character constants:**  
- Examples: `'a'`, `'\\n'`  
- Type: `int` (ASCII value)  
- Supports escape sequences (`\n`, `\t`, `\\`, etc.)  

**Strings:**  
- Example: `"hello"`  
- Automatically null-terminated (`\0`)  
- Type: `string`  

---

## 2.4 Arithmetic Operators

| Operator | Meaning        |
|-----------|----------------|
| `+`       | addition       |
| `-`       | subtraction    |
| `*`       | multiplication |
| `/`       | division       |
| `%`       | modulus        |

- Works only on integers and pointers  
- Division by zero → runtime error  
- No overflow/underflow detection  
- Use parentheses `()` for precedence control  

---

## 2.5 Relational Operators

| Operator | Meaning     |
|-----------|-------------|
| `<`, `<=`, `>`, `>=` | comparison |
| `==`, `!=` | equality |

- Returns `1` (true) or `0` (false)  
- Can compare integers, pointers, or strings (similar to `strcmp()`)  

---

## 2.6 Logical Operators

| Operator | Meaning |
|-----------|----------|
| `&&` | AND |
| `||` | OR |
| `^^` | XOR |
| `!` | NOT (unary) |

- `&&` and `||` use short-circuit evaluation  
- Operands: integers or pointers (non-zero = true)  

---

## 2.7 Bitwise Operators

| Operator | Meaning |
|-----------|----------|
| `&` | AND |
| `|` | OR |
| `^` | XOR |
| `<<` | left shift |
| `>>` | right shift |
| `~` | bitwise NOT (unary) |

- Shifting signed values → arithmetic shift  
- Shifting by invalid counts → undefined behavior (compiler error if detectable)  

---

## 2.8 Assignment Operators

| Operator | Example | Meaning |
|-----------|----------|----------|
| `=` | `x = y` | assign |
| `+=` | `x += y` | add and assign |
| `-=` | `x -= y` | subtract and assign |
| `*=` | `x *= y` | multiply and assign |
| `/=` | `x /= y` | divide and assign |
| `%=` | `x %= y` | mod and assign |
| `&=`, `|=`, `^=`, `<<=`, `>>=` | bitwise & shift assignments |

- Only modifiable variables/arrays can be assigned  
- Each assignment returns the new value of the left operand  

---

## 2.9 Increment / Decrement

Operators: `++` and `--` (prefix or postfix).  
- If undeclared → implicitly becomes `int64_t`  
- Works on integers or pointers  
  - Integer → ±1  
  - Pointer → ±(size of type pointed to)  

---

## 2.10 Conditional Expressions

**Syntax:**  
```c
x = i == 0 ? "zero" : "non-zero";

