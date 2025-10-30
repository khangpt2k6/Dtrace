# Pointers and Arrays in DTrace
## **1. Pointers in DTrace**

* **Definition**: Pointers are memory addresses of data objects in either kernel or user process memory.

* **Syntax**: Similar to C/C++.

  ```d
  int *p;  // p is a pointer to an integer
  ```

* **Operators**:

  * `&` → get address of an object
  * `*` → dereference a pointer to access the value

  ```d
  p = &`kmem_flags;  // address of kmem_flags
  trace(*p);          // value at that address
  ```

* **Pointer Safety**:

  * Invalid pointer access does **not crash DTrace** or the OS.
  * Errors (invalid address or alignment errors) are reported to the user.
  * Example of invalid pointer:

    ```d
    x = (int *)NULL;
    y = *x;  // triggers DTrace error
    ```

---

## **2. Arrays**

* **Scalar Arrays**: Fixed-length arrays stored consecutively in memory.

  ```d
  int a[5];  // array of 5 integers
  a[0] = 10; // first element
  ```

* **Associative Arrays**: Dynamic, key-value storage, non-consecutive memory.

  ```d
  int a[int];
  a[0] = 10;
  a[-5] = 20;
  ```

* **Array vs Pointer**:

  * Array name = address of first element → can assign to a pointer:

    ```d
    int *p;
    p = a;   // equivalent to p = &a[0]
    ```
  * Cannot assign arrays to each other directly.

* **Multi-Dimensional Arrays**:

  ```d
  int a[12][34];  // 12 rows, 34 columns
  a[0][1]         // access row 0, col 1
  ```

---

## **3. Pointer & Array Relationship**

* Pointer arithmetic matches array indexing:

  ```d
  p = &a[0];
  trace(p[2]);  // same as trace(a[2]);
  ```
* Pointer arithmetic adjusts by the size of the type:

  ```d
  int *x;
  x = &a[0];
  trace(x);     // 0
  trace(x + 1); // 4 if int = 4 bytes
  ```
* Subtracting pointers gives the number of elements between them:

  ```d
  y - x; // difference in array indices
  ```

---

## **4. Generic Pointers**

* **void*** → pointer without a specified type; cannot dereference or do arithmetic.
* **uintptr_t** → integer type for storing pointer values.
* Use casts for conversions:

  ```d
  void *vp;
  int *ip = (int *)vp;   // cast to specific type to dereference
  uintptr_t u = (uintptr_t)ip;  // cast to integer
  ```

---

## **5. Pointer Restrictions in DTrace**

* Cannot take addresses of:

  * Associative arrays
  * Built-in functions
  * DTrace variables
* Cannot perform indirect function calls or assignments using pointers.
* Only assign or access values directly or via `[ ]` operator.

---

## **6. Address Spaces**

* DTrace runs in **kernel address space**, but user processes have separate virtual memory.
* **Do not dereference user pointers directly** (from system call args, etc.).
* Use `copyin`, `copyinstr`, or `copyinto` to safely access user memory.
* Store user addresses as `uintptr_t` to avoid accidental kernel dereference.

