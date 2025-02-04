# Chapter 25: Unsafe Rust

Rust is widely recognized for its strong safety guarantees. By leveraging compile-time static analysis and runtime checks (such as array bounds checking), it prevents many common memory and concurrency bugs. However, Rust's static analysis is conservative—it might reject code that is actually safe if it cannot prove that all invariants are met. Moreover, hardware itself is inherently unsafe, and low-level systems programming often requires direct interaction with hardware. To support such programming while preserving as much safety as possible, Rust provides **Unsafe Rust**.

Unsafe Rust is not a separate language but an extension of safe Rust. It grants access to certain operations that safe Rust disallows. In exchange for this power, you must manually uphold Rust's core safety invariants. Many parts of the standard library, such as slice manipulation functions, vector internals, and thread and I/O management, are implemented as safe abstractions over unsafe code. This pattern—isolating unsafe code behind a safe API—is crucial for maintaining overall program safety.

----

## 25.1 Overview of Unsafe Rust

### 25.1.1 What Is Unsafe Rust?

In safe Rust, the compiler prevents problems like data races, invalid memory access, and dangling pointers. However, there are situations where the compiler cannot confirm that an operation is safe—even if the operation is correct when used carefully. This is when **unsafe Rust** comes into play.

Unsafe Rust permits five operations that safe Rust forbids:

1. **Dereferencing raw pointers** (`*const T` and `*mut T`).
2. **Calling unsafe functions** (including foreign C functions).
3. **Accessing and modifying mutable static variables**.
4. **Implementing unsafe traits**.
5. **Accessing union fields**.

Aside from these operations, Rust's usual rules regarding ownership, borrowing, and type checking still apply.
Unsafe Rust does not turn off all safety checks. It only relaxes restrictions on the five operations listed above.

### 25.1.2 Why Do We Need Unsafe Code?

Rust is designed to support low-level systems programming while maintaining high safety standards. Nevertheless, certain scenarios demand unsafe code:

- **Hardware Interaction**: Accessing memory-mapped I/O or device registers requires direct hardware manipulation, which is inherently unsafe.  
- **Foreign Function Interface (FFI)**: Interacting with C or other languages that do not preserve Rust's safety invariants.  
- **Advanced Data Structures**: Implementing intrusive linked lists or lock-free structures may involve operations not expressible in safe Rust.  
- **Performance Optimizations**: Specialized optimizations can require pointer arithmetic or custom memory layouts that exceed safe abstractions.

Because the compiler cannot verify correctness in these contexts, you must manually ensure that the code maintains all necessary safety properties.

----

## 25.2 Unsafe Blocks and Unsafe Functions

Rust only permits unsafe operations within blocks or functions explicitly marked with the `unsafe` keyword.

### 25.2.1 Declaring an Unsafe Block

An *unsafe block* is a code block prefixed with `unsafe`, intended for operations the compiler cannot verify as safe.

A key use of an unsafe block is **dereferencing raw pointers**.
Raw pointers in Rust are similar to C pointers and are discussed in detail in the next section. Creating a raw pointer is safe, but dereferencing it is unsafe, because the compiler cannot ensure the pointer is valid. The `unsafe { ... }` block explicitly indicates that the programmer is taking responsibility for memory safety.

In the example below, we define a mutable raw pointer using `*mut`. Dereferencing it is allowed only inside an unsafe block:

```rust,editable
fn main() {
    let mut num: i32 = 42;
    let r: *mut i32 = &mut num; // Create a raw mutable pointer to num

    unsafe {
        *r = 99; // Dereference and modify the value using the raw pointer
        println!("The value of num is: {}", *r);
    }
}
```

**Explanation:**
- We create a raw mutable pointer `r` that points to `num`.
- Inside an `unsafe` block, we dereference `r` and modify the value.

Although this example is safe in practice, that's because `r` comes from a valid reference and remains in the same function where it was created.

### 25.2.2 Declaring an Unsafe Function

You can mark a function with `unsafe` if its correct usage depends on the caller upholding certain invariants that Rust cannot verify. Within an unsafe function, both safe and unsafe code can be used freely, but any call to such a function must occur in an unsafe block.

```rust,editable
unsafe fn dangerous_function(ptr: *const i32) -> i32 {
    // Dereferencing a raw pointer is allowed here.
    *ptr
}

fn main() {
    let x = 42;
    let ptr = &x as *const i32;
    // Any call to an unsafe function must be wrapped in an unsafe block.
    unsafe {
        println!("Value: {}", dangerous_function(ptr));
    }
}
```

Here, `unsafe` indicates that this function has certain requirements the caller must satisfy, e.g. passing only valid pointers to i32 instances.
Calling it inside an unsafe block implies you've read the function's documentation and will ensure all invariants are upheld.

### 25.2.3 Unsafe Block or Unsafe Function?

When choosing between using an **unsafe block** or marking a function as **unsafe**, focus on the function's contract more than whether it includes unsafe code:

- **Use `unsafe fn`** if misuse (while still compiling) could cause undefined behavior. In other words, the function inherently requires the caller to uphold a particular safety contract.  
- **Keep the function safe** if no well-typed call can result in undefined behavior. Even if the function body contains an `unsafe` block, that block may internally uphold all the necessary guarantees.

Avoid marking a function as `unsafe` solely because it contains an `unsafe` block—doing so might mislead callers into believing there are extra safety concerns. In general, opt for an **unsafe block** unless you truly need an unsafe function contract.

A common approach is to encapsulate unsafe code within a safe function, exposing a safe API and confining unsafe code to a small scope.

----

## 25.3 Raw Pointers in Rust

Rust provides two forms of raw pointers:

- `*const T` — a pointer to a constant `T` (read-only).
- `*mut T` — a pointer to a mutable `T`.

The `*` here is part of the type name, signifying an immutable or mutable raw pointer; there is no `*T` type without `const` or `mut`.

Raw pointers enable unrestricted memory access and allow you to construct data structures that Rust's type system would typically forbid.

### 25.3.1 Creating vs. Dereferencing Raw Pointers

You can create raw pointers by casting references, and you dereference them with the `*` operator. Although Rust may automatically dereference safe references, it does not do so with raw pointers.

- **Creating, passing around, or comparing raw pointers** is safe.  
- **Dereferencing a raw pointer** to read or write memory is unsafe.

Other pointer operations, like adding an offset, can be safe or unsafe: for example, `ptr.add()` is considered unsafe, while `ptr.wrapping_add()` is regarded as safe, even though it can produce an invalid address.

For example:

```rust,editable
fn increment_value_by_pointer() {
    let mut value = 10;
    // Converting a mutable reference to a raw pointer is safe.
    let value_ptr = &mut value as *mut i32;
    
    // Dereferencing the raw pointer to modify the value is unsafe.
    unsafe {
        *value_ptr += 1;
        println!("The incremented value is: {}", *value_ptr);
    }
}

fn dereference_raw_pointers() {
    let mut num = 5;
    let r1 = &num as *const i32;
    let r2 = &mut num as *mut i32;

    // Potentially invalid raw pointers:
    let invalid0 = &mut 0 as *const i32;      // Points to a temporary
    let invalid1 = &mut 123456 as *const i32; // Arbitrary invalid address
    let invalid2 = &mut 0xABCD as *mut i32;   // Also invalid

    unsafe {
        println!("r1 is: {}", *r1);
        println!("r2 is: {}", *r2);
        // Dereferencing invalid0, invalid1, or invalid2 here would be undefined behavior.
    }
}

fn main() {
    increment_value_by_pointer();
    dereference_raw_pointers();
}
```

Because the pointers `r1` and `r2` come from valid references, we assume they are safe to dereference. This assumption does not hold for arbitrary raw pointers. Simply owning an invalid pointer is not immediately dangerous, but dereferencing it is undefined behavior.

### 25.3.2 Pointer Arithmetic

Raw pointers permit arithmetic similar to that in C. For example, you can move a pointer forward by a certain number of elements in an array:

```rust,editable
fn pointer_arithmetic_example() {
    let arr = [10, 20, 30, 40, 50];
    let ptr = arr.as_ptr(); // A raw pointer to the array

    unsafe {
        // Move the pointer forward by 2 elements (not bytes).
        let third_ptr = ptr.add(2);
        println!("The third element is: {}", *third_ptr);
    }
}

fn main() {
    pointer_arithmetic_example();
}
```

Because `ptr.add(2)` bypasses Rust's safety checks for bounds and memory layout, using it is inherently unsafe. For more details on raw pointers, see [Pointers](https://doc.rust-lang.org/std/primitive.pointer.html).

### 25.3.3 Fat Pointers

A raw pointer to an unsized type is referred to as a **fat pointer**, just like an equivalent reference or `Box`. For instance, `*const [i32]` contains both the address and the slice's length.

----

## 25.4 Memory Handling in Unsafe Code

Even within unsafe blocks, Rust's ownership model and RAII (Resource Acquisition Is Initialization) still apply. For instance, if you allocate a `Vec<T>` inside an unsafe block, it will be deallocated automatically when it goes out of scope.

Nevertheless, unsafe code can circumvent some of Rust's normal safety checks. When using unsafe features, you must ensure the following:

- **No data races** occur when multiple threads share memory.  
- **Memory safety** is upheld (e.g., do not dereference pointers to freed memory, avoid double frees, and do not perform invalid deallocations).

----

## 25.5 Casting and `std::mem::transmute`

Safe Rust allows only a limited set of casts (for instance, certain integer-to-integer conversions). However, if you need to reinterpret a type's bits as another type, you must employ unsafe features.

Two key mechanisms are:

1. The `as` operator, which covers some conversions but not all.  
2. `std::mem::transmute`, which reinterprets the bits of a value as a different type with no runtime checks.

`transmute` is effectively a bitwise copy between types. You must specify the source and destination types, and they must match in size. If they do not, the compiler will reject the code (unless you use nightly features to override this, which is highly unsafe).

### 25.5.1 Example: Reinterpreting Bits with `transmute`

```rust,editable
fn float_to_bits(f: f32) -> u32 {
    unsafe { std::mem::transmute::<f32, u32>(f) }
}

fn bits_to_float(bits: u32) -> f32 {
    unsafe { std::mem::transmute::<u32, f32>(bits) }
}

fn main() {
    let f = 3.14f32;
    let bits = float_to_bits(f);
    println!("Float: {}, bits: 0x{:X}", f, bits);

    let f2 = bits_to_float(bits);
    println!("Back to float: {}", f2);
}
```

Because `transmute` reinterprets bits without checking types, incorrect usage can quickly lead to undefined behavior. Often, safer options (like the built-in `to_bits` method for floats) are more appropriate.

----

## 25.6 Calling C Functions (FFI)

One of the most common uses of unsafe Rust is calling C libraries through the Foreign Function Interface (FFI). In an `extern "C"` block, you declare the external functions you want to call. The `"C"` indicates the application binary interface (ABI), telling Rust how to call these functions at the assembly level. You also use the `#[link(...)]` attribute to specify which libraries to link.

```rust,editable
#[link(name = "c")]
extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    let value = -42;
    // Calling an external fn is unsafe because Rust cannot verify its implementation.
    unsafe {
        let result = abs(value);
        println!("abs({}) = {}", value, result);
    }
}
```

When you declare the argument types for a foreign function, the Rust compiler cannot verify that your declarations match the function's real signature. A mismatch can produce undefined behavior.

### 25.6.1 Providing Safe Wrappers

A common practice is to expose a safe abstraction over unsafe functionality:

```rust,editable
#[link(name = "c")]
extern "C" {
    fn abs(input: i32) -> i32;
}

fn safe_abs(value: i32) -> i32 {
    unsafe { abs(value) }
}

fn main() {
    println!("abs(-5) = {}", safe_abs(-5));
}
```

This approach confines the unsafe parts of your code to a small area, creating a clearer, safer API.

----

## 25.7 Rust Unions

Rust unions resemble C unions, enabling multiple fields to share the same underlying memory. Unlike Rust enums, unions do not track which variant is currently active, so accessing union fields is inherently unsafe.

```rust
union MyUnion {
    int_val: u32,
    float_val: f32,
}

fn union_example() {
    let u = MyUnion { int_val: 0x41424344 };
    unsafe {
        // Reading from a union field reinterprets the bits.
        println!("int: 0x{:X}, float: {}", u.int_val, u.float_val);
    }
}
```

Since the compiler does not know which field is valid at any point, you must ensure you only read the field that was last written. Otherwise, you risk undefined behavior.

----

## 25.8 Mutable Global Variables

Global mutable variables in Rust are declared with `static mut`. They are inherently unsafe because concurrent or multi-path write access can lead to data races.

```rust
static mut COUNTER: i32 = 0;

fn increment() {
    unsafe {
        COUNTER += 1;
    }
}
```

It's best to minimize the use of mutable globals. When they are required, you should apply synchronization primitives to ensure safe, race-free access.

----

## 25.9 Unsafe Traits

Some traits in Rust are marked `unsafe` if an incorrect implementation can cause undefined behavior. This is especially common for traits involving pointer aliasing, concurrency, or similar low-level operations that the compiler cannot fully verify.

```rust,ignore
unsafe trait MyUnsafeTrait {
    // Methods or invariants that the implementer must maintain.
}

struct MyType;

unsafe impl MyUnsafeTrait for MyType {
    // Implementation that respects the trait's invariants.
}
```

Implementing an unsafe trait is a serious responsibility. Violating the trait's contract can undermine assumptions that other code relies on for safety.

----

## 25.10 Example: Splitting a Mutable Slice (`split_at_mut`)

A classic example in the standard library is the `split_at_mut` function, which splits a mutable slice into two non-overlapping mutable slices. Safe Rust does not allow creating two mutable slices from one slice, because the compiler cannot prove that the slices do not overlap. The example below uses unsafe functions (like `std::slice::from_raw_parts_mut`) and pointer arithmetic to provide this functionality:

```rust,editable
fn my_split_at_mut(slice: &mut [u8], mid: usize) -> (&mut [u8], &mut [u8]) {
    let len = slice.len();
    assert!(mid <= len);
    let ptr = slice.as_mut_ptr();
    unsafe {
        (
            std::slice::from_raw_parts_mut(ptr, mid),
            std::slice::from_raw_parts_mut(ptr.add(mid), len - mid),
        )
    }
}

fn main() {
    let mut data = [1, 2, 3, 4, 5];
    let (left, right) = my_split_at_mut(&mut data, 2);
    left[0] = 42;
    right[0] = 99;
    println!("{:?}", data); // Outputs: [42, 2, 99, 4, 5]
}
```

By carefully ensuring that the two returned slices do not overlap, the function safely exposes low-level pointer arithmetic in a high-level, safe API.

----

## 25.11 Tools for Verifying Unsafe Code

Even with diligent reviews, unsafe code can still harbor memory errors. A valuable tool for detecting such issues is **[Miri](https://github.com/rust-lang/miri)**—an interpreter for Rust code that can discover undefined behavior, including:

- Out-of-bounds memory access  
- Use-after-free errors  
- Invalid deallocation  
- Data races in single-threaded contexts (e.g., dereferencing freed memory)

Another well-known tool for detecting memory-related issues is **Valgrind**, which can be used with Rust binaries as well.

### 25.11.1 Installing and Using Miri

Depending on your operating system, Miri may already be installed alongside other Rust tools, or it can be installed via Rustup:

1. **Install Miri** (if necessary):
   ```bash
   rustup component add miri
   ```

2. **Run Miri** on your tests:
   ```bash
   cargo miri test
   ```

Miri interprets your code and flags invalid memory operations, helping you verify that your unsafe code is correct. It can even detect memory leaks caused by cyclic data structures in safe Rust.

----

## 25.12 Example: A Bug Miri Might Catch

Consider a function that returns a pointer to a local variable:

```rust,ignore
fn return_dangling_pointer() -> *const i32 {
    let x = 10;
    &x as *const i32
}

fn main() {
    let ptr = return_dangling_pointer();
    unsafe {
        // Danger: 'x' is out of scope, so dereferencing 'ptr' is undefined behavior.
        println!("Value is {}", *ptr);
    }
}
```

Although this code might occasionally print `10` and appear to work, it exhibits undefined behavior. The pointer refers to memory that is no longer valid. Tools like Miri can detect this mistake before it leads to more serious problems.

----

## 25.13 Inline Assembly

Rust supports inline assembly for cases where you need direct control over the CPU or hardware—important in some low-level systems programming tasks. You use the `asm!` macro (from `std::arch`), and it must live inside an unsafe block because the compiler cannot validate the assembly's correctness or safety.

### 25.13.1 When and Why to Use Inline Assembly

Inline assembly is useful for:

- **Performance-Critical Operations**: Certain optimizations might require instructions the compiler does not normally emit.  
- **Hardware Interaction**: For instance, manipulating CPU registers or interfacing with specialized hardware.  
- **Low-Level Algorithms**: Some algorithms need unusual instructions or extra tuning.

### 25.13.2 Using Inline Assembly

The `asm!` macro specifies the assembly instructions, input/output operands, and additional options. Below is a simple example (on x86_64) that moves an immediate value into a variable:

```rust,editable
use std::arch::asm;

fn main() {
    let mut x: i32 = 0;
    unsafe {
        // Moves the immediate value 5 into the register bound to 'x'.
        asm!("mov {0}, 5", out(reg) x);
    }
    println!("x is: {}", x);
}
```

- `mov {0}, 5` loads the literal 5 into the register bound to `x`.
- `out(reg) x` places the result in `x` after the assembly finishes.
- The entire block is `unsafe` because the compiler cannot check assembly code.

### 25.13.3 Best Practices and Considerations

- **Encapsulation**: Keep inline assembly in small functions or modules, exposing a safe API if possible.  
- **Platform Specifics**: Inline assembly is architecture-specific; code written for x86_64 may not work on other platforms.  
- **Stability**: Inline assembly might require nightly Rust on certain targets or for specific features.  
- **Documentation**: Explain the rationale and assumptions behind any assembly so that future maintainers understand its safety requirements.

Used sparingly and carefully, inline assembly in unsafe blocks gives you fine-grained control while preserving most of Rust's safety guarantees elsewhere.

----

## 25.14 Summary and Further Resources

Unsafe Rust allows you to step outside the bounds of safe Rust, enabling low-level programming and direct hardware interaction. However, with these capabilities come responsibilities: you must manually guarantee memory safety, freedom from data races, and other critical invariants.

In this chapter, we covered:

- **The Nature of Unsafe Rust**: Its definition, the five operations it enables, and why Rust needs it.  
- **Reasons for Unsafe Code**: Hardware interaction, FFI, advanced data structures, and performance optimizations.  
- **Unsafe Blocks and Functions**: How to create them properly, including the requirement to call unsafe functions in unsafe blocks.  
- **Raw Pointers**: Their creation, dereferencing, pointer arithmetic, and how they differ from safe references.  
- **Casting and `transmute`**: How to reinterpret memory at a bit level, with an emphasis on the associated dangers.  
- **Memory Handling**: Interactions with RAII and the pitfalls of data races and invalid deallocations.  
- **FFI**: How to declare and call external C functions, and how to wrap them in safe functions.  
- **Unions and Mutable Globals**: Their uses, how they differ from typical variables, and their inherent dangers.  
- **Unsafe Traits**: Why some traits are unsafe and what it means to implement them.  
- **Examples**: Such as using unsafe pointer arithmetic to split a mutable slice.  
- **Verification Tools**: How to use Miri to detect undefined behavior in unsafe code.  
- **Inline Assembly**: Employing the `asm!` macro for direct CPU or hardware interactions.

### 25.14.1 Best Practices for Using Unsafe Code

- **Prefer Safe Rust**: Rely on safe abstractions whenever possible.  
- **Localize Unsafe Code**: Confine unsafe operations to small blocks or modules that can be thoroughly reviewed.  
- **Document Invariants**: Make explicit the assumptions and requirements that the unsafe code depends on.  
- **Review and Test**: Use tools like Miri and perform rigorous code reviews.

### 25.14.2 Further Reading

- The [**Rustonomicon**](https://doc.rust-lang.org/stable/nomicon/) for an in-depth exploration of advanced unsafe topics.  
- [**Rust Atomics and Locks**](https://marabos.nl/atomics/) by Mara Bos, a comprehensive low-level resource on concurrency.  
- **Programming Rust** by Jim Blandy, Jason Orendorff, and Leonora F.S. Tindall provides a detailed discussion of unsafe Rust with examples for its use.

Used judiciously, unsafe Rust offers the kind of low-level control found in C, while retaining Rust's safety benefits in the majority of your code.
