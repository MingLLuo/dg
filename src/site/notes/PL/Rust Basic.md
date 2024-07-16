---
{"dg-publish":true,"permalink":"/pl/rust-basic/","created":"2024-07-16T00:57:20.965+08:00","updated":"2024-07-16T13:31:07.611+08:00"}
---


#Rust
### Built-in Types
#### Scalar Types
A _scalar_ type represents a single value.

|                        | Types                                      | Literals                       |
| ---------------------- | ------------------------------------------ | ------------------------------ |
| Signed integers        | `i8`, `i16`, `i32`, `i64`, `i128`, `isize` | `-10`, `0`, `1_000`, `123_i64` |
| Unsigned integers      | `u8`, `u16`, `u32`, `u64`, `u128`, `usize` | `0`, `123`, `10_u16`           |
| Floating point numbers | `f32`, `f64`                               | `3.14`, `-10.0e20`, `2_f32`    |
| Unicode scalar values  | `char`                                     | `'a'`, `'α'`, `'∞'`            |
| Booleans               | `bool`                                     | `true`, `false`                |

The types have widths as follows:
- `iN`, `uN`, and `fN` are _N_ bits wide,
- `isize` and `usize` are the width of a pointer,
- `char` is 32 bits wide,
- `bool` is 8 bits wide.

Additionally, the `isize` and `usize` types depend on the architecture of the computer your program is running on, which is denoted in the table as "arch": 64 bits if you’re on a 64-bit architecture and 32 bits if you’re on a 32-bit architecture.

> Some people will be confused by machine's architecture bit size, which refers to the memory addressing length(the length of pointers). You can directly use 64-bits int in a 32-bits machine, with minor discount in performance. The compiler may need to generate several machine code instructions to perform operations on the 64 bit values, slowing down those operations by several times.


For float point number, rust implement related literal like `INFINITY`, `NEG_INFINITY`, `NAN`, `MIN`, `MAX`
- https://www.gnu.org/software/libc/manual/html_node/Infinity-and-NaN.html

#### Compound Types
- Tuple
- Array/Vec/Slice
- Struct
- Enum
- Pointer
	- Reference
	- Box
	- Bare Pointer
- String

### Ownership

- The concept of copying the pointer, length, and capacity without copying the data probably sounds like making a shallow copy. But because Rust also invalidates the first variable, instead of being called a shallow copy, it’s known as a _move_
- Followed by an important trait `Copy`. If a type implements the `Copy` trait, variables that use it do not move, but rather are trivially copied, making them still valid after assignment to another variable. Here are some of the types that implement `Copy`:
	- All the integer types, such as `u32`.
	- The Boolean type, `bool`, with values `true` and `false`.
	- All the floating-point types, such as `f64`.
	- The character type, `char`.
	- Tuples, if they only contain types that also implement `Copy`. For example, `(i32, i32)` implements `Copy`, but `(i32, String)` does not.
- `Rc` and `Arc` can share the ownership of the variable, `A` means support with `atomic`. It's clear that `Rc` provides faster counter update, while `Arc` gives us more security update guarantee.
### Reference
- [Do I need to have 64 bit Processor to use 64 bit data type](https://stackoverflow.com/questions/5530906/do-i-need-to-have-64-bit-processor-to-use-64-bit-data-type)
- [That’s Not Normal–the Performance of Odd Floats](https://randomascii.wordpress.com/2012/05/20/thats-not-normalthe-performance-of-odd-floats/)