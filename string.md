# D Language — Strings
---
## **1. String Representation**

* Strings in DTrace are **null-terminated character arrays** (`\0`).
* Stored in **fixed-size arrays** for consistent tracing; default max length is **256 bytes**, configurable via `strsize`.
* **D string type** exists separately from `char *` to simplify tracing:

  ```d
  trace(s);  // if s is a string, prints the full null-terminated string
  trace(*s); // if s is char *, prints the first character
  ```

---

## **2. String Constants**

* Declared with **double quotes `" "`**, automatically terminated with `\0`.
* Cannot contain literal newlines; use `\n` instead.
* Special escape sequences supported: `\t`, `\n`, `\"`, etc.
* Size = number of characters + 1 (for null byte).

---

## **3. String Assignment**

* **Copied by value**, not by reference (unlike `char *`).

  ```d
  s = "hello";  // copies 6 bytes ("hello" + null byte)
  ```
* Compatible types like `char *` or `char[n]` are automatically promoted to string.
* If source exceeds destination limit → string truncated.

---

## **4. String Conversion**

* Convert other expressions to string using:

  ```d
  s = (string) expression;
  s = stringof(expression);
  ```
* Scalar types, pointers, or scalar array addresses can be converted.
* Invalid addresses are safely handled by DTrace, but may produce garbage output.

---

## **5. String Comparison**

* **Relational operators** (`<`, `<=`, `>`, `>=`, `==`, `!=`) work on strings.
* Comparison is **byte-by-byte**, based on ASCII values, until null byte or max string length.
* Returns **int**: `1` = true, `0` = false.

**Examples:**

```d
"coffee" < "espresso"   // 1 (true)
"coffee" == "coffee"    // 1 (true)
"coffee" >= "mocha"     // 0 (false)
```
