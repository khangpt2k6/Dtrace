# D Language — Types, Operators, and Expressions

---

## 2.1 Identifier Names & Keywords

### Identifiers

Composed of letters, digits, and underscores (`_`)
- **Must start with:** letter or `_`
- **Reserved:** Names starting with `_`

### Naming Conventions

- **`MixedCase`** → variables
- **`UPPERCASE`** → constants

### Keywords

- Superset of ANSI-C keywords
- `*` → reserved for future use
- `+` → D-specific (e.g., `self`, `translator`, `xlate`)
- ⚠️ No control-flow constructs (`if`, `for`, etc.) allowed yet

![D Language Keywords](https://github.com/user-attachments/assets/9a15aa6c-7613-46a8-a7cc-66c88fc71c09)

---

## 2.2 Data Types and Sizes

> **Note:** Arithmetic is only allowed on **integers** (not floating-point types)

**Data model:** 32-bit or 64-bit (check via `isainfo -b`)

### Integer Types

| Type         | 32-bit  | 64-bit  |
|--------------|---------|---------|
| `char`       | 1 byte  | 1 byte  |
| `short`      | 2 bytes | 2 bytes |
| `int`        | 4 bytes | 4 bytes |
| `long`       | 4 bytes | 8 bytes |
| `long long`  | 8 bytes | 8 bytes |

**Properties:**
- Default: **signed** (unless specified as `unsigned`)
- Type aliases available: `int8_t`, `uint32_t`, `intptr_t`, etc.

### Floating-Point Types

| Type           | Size     |
|----------------|----------|
| `float`        | 4 bytes  |
| `double`       | 8 bytes  |
| `long double`  | 16 bytes |

**Limitations:**
- ❌ No floating-point arithmetic
- ✅ Only tracing and formatting supported

### Special Types

- **`string`** → special type for ASCII strings

---

## 2.3 Constants

### Integer Constants

**Formats:**
- **Decimal:** `123`
- **Octal:** `0123`
- **Hexadecimal:** `0x123`

**Suffixes:**

| Suffix        | Type            |
|---------------|-----------------|
| `u` / `U`     | unsigned        |
| `l` / `L`     | long            |
| `ll` / `LL`   | long long       |
| `ul` / `UL`   | unsigned long   |
| `ull` / `ULL` | unsigned long long |

### Floating-Point Constants

- **Forms:** decimal or exponent notation
- **Default type:** `double`
- **Suffixes:**
  - `f` / `F` → `float`
  - `l` / `L` → `long double`

### Character Constants

**Examples:** `'a'`, `'\n'`

- **Type:** `int` (ASCII value)
- **Escape sequences:** `\n`, `\t`, `\\`, `\0`, etc.

### String Constants

**Example:** `"hello"`

- **Type:** `string`
- Automatically null-terminated with `\0`

![Escape Sequences Reference](https://github.com/user-attachments/assets/5f6c3a78-3e15-4987-a3d2-34e21a628f53)

---

## 2.4 Arithmetic Operators

| Operator | Operation      |
|----------|----------------|
| `+`      | Addition       |
| `-`      | Subtraction    |
| `*`      | Multiplication |
| `/`      | Division       |
| `%`      | Modulus        |

**Restrictions:**
- ✅ Works only on **integers** and **pointers**
- ⚠️ Division by zero → **runtime error**
- ⚠️ No overflow/underflow detection
- 💡 Use parentheses `()` for precedence control

---

## 2.5 Relational Operators

| Operator | Meaning              |
|----------|----------------------|
| `<`      | Less than            |
| `<=`     | Less than or equal   |
| `>`      | Greater than         |
| `>=`     | Greater than or equal|
| `==`     | Equal to             |
| `!=`     | Not equal to         |

**Returns:** `1` (true) or `0` (false)

**Can compare:**
- Integers
- Pointers
- Strings (similar to `strcmp()`)

---

## 2.6 Logical Operators

| Operator | Operation |
|----------|-----------|
| `&&`     | AND       |
| `\|\|`   | OR        |
| `^^`     | XOR       |
| `!`      | NOT       |

**Features:**
- `&&` and `||` use **short-circuit evaluation**
- **Operands:** integers or pointers (non-zero = true)

---

## 2.7 Bitwise Operators

| Operator | Operation      |
|----------|----------------|
| `&`      | AND            |
| `\|`     | OR             |
| `^`      | XOR            |
| `<<`     | Left shift     |
| `>>`     | Right shift    |
| `~`      | NOT (unary)    |

**Notes:**
- Shifting signed values → **arithmetic shift**
- Invalid shift counts → **undefined behavior** (compiler error if detectable)

---

## 2.8 Assignment Operators

| Operator | Example   | Meaning              |
|----------|-----------|----------------------|
| `=`      | `x = y`   | Assign               |
| `+=`     | `x += y`  | Add and assign       |
| `-=`     | `x -= y`  | Subtract and assign  |
| `*=`     | `x *= y`  | Multiply and assign  |
| `/=`     | `x /= y`  | Divide and assign    |
| `%=`     | `x %= y`  | Modulus and assign   |
| `&=`     | `x &= y`  | Bitwise AND assign   |
| `\|=`    | `x \|= y` | Bitwise OR assign    |
| `^=`     | `x ^= y`  | Bitwise XOR assign   |
| `<<=`    | `x <<= y` | Left shift assign    |
| `>>=`    | `x >>= y` | Right shift assign   |

**Rules:**
- Only modifiable variables/arrays can be assigned
- Each assignment returns the new value of the left operand

---

## 2.9 Increment / Decrement

**Operators:** `++` and `--` (prefix or postfix)

**Behavior:**
- If variable undeclared → implicitly becomes `int64_t`
- Works on integers or pointers
  - **Integer:** ± 1
  - **Pointer:** ± (size of type pointed to)

---

## 2.10 Conditional Expressions

**Syntax:**
```c
x = i == 0 ? "zero" : "non-zero";
```

**Rules:**
- Evaluates the first condition
- Chooses second expression if true, third if false
- Result types must be **compatible** (both strings or both integers)

---

## 2.11 Type Conversions

Follows **"usual arithmetic conversions"** (ANSI-C rules)

### Rank Order

```
char < short < int < long < long long
```

- Unsigned types rank slightly higher than signed counterparts
- Lower-ranked operand promoted to higher-ranked type

### Explicit Casting

**Syntax:**
```c
y = (int)x;
```

**Behavior:**
- **Signed types:** sign extension
- **Unsigned types:** zero-fill
- ❌ No float arithmetic → no float conversions

---

## 2.12 Operator Precedence & Associativity

| Precedence | Operators | Associativity |
|------------|-----------|---------------|
| **Highest** | `()` `[]` `->` `.` | Left |
| | `!` `~` `++` `--` `+` `-` `*` `&` `(type)` `sizeof` `stringof` `offsetof` `xlate` | Right |
| | `*` `/` `%` | Left |
| | `+` `-` | Left |
| | `<<` `>>` | Left |
| | `<` `<=` `>` `>=` | Left |
| | `==` `!=` | Left |
| | `&` | Left |
| | `^` | Left |
| | `\|` | Left |
| | `&&` | Left |
| | `^^` | Left |
| | `\|\|` | Left |
| | `?:` | Right |
| | `=` `+=` `-=` `*=` `/=` `%=` `&=` `\|=` `^=` `<<=` `>>=` | Right |
| **Lowest** | `,` | Left |

### Operator Categories

- **`()`** → Function calls
- **`[]`** → Array or associative array access
- **`->` `.`** → Struct/union member access
- **`,`** → Expression sequencing (evaluation order not guaranteed)

---

> **📚 Reference:** D Language Specification - Types, Operators, and Expressions
