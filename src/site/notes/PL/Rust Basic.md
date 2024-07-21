---
{"dg-publish":true,"permalink":"/pl/rust-basic/","noteIcon":"","created":"2024-07-16T00:57:20.965+08:00","updated":"2024-07-20T00:49:26.669+08:00"}
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

### Struct
- Everything within this `impl` block will be associated with specific type.
- We call `rect1.area()` is a method syntax calling `area` method on `Rectangle` instance.
```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}


fn main () {
	let rect1 = {
		width: 10,
		height: 20,
	};
	println!( 
		"The area of the rectangle is {} square pixels.",
		rect1.area() 
	);
}
```

>Rust doesn’t have an equivalent to the `->` operator; instead, Rust has a feature called _automatic referencing and dereferencing_. Calling methods is one of the few places in Rust that has this behavior.

- Here’s how it works: when you call a method with `object.something()`, Rust automatically adds in `&`, `&mut`, or `*` so `object` matches the signature of the method. In other words, the following are the same:
	- `p1.distance(&p2); (&p1).distance(&p2);`
The first one looks much cleaner. This automatic referencing behavior works because methods have a clear receiver—the type of `self`. Given the receiver and name of a method, Rust can figure out definitively whether the method is reading (`&self`), mutating (`&mut self`), or consuming (`self`). The fact that Rust makes borrowing implicit for method receivers is a big part of making ownership ergonomic in practice.

### Enum
- We call `V4` and `V6` the variants of the enum
```rust
enum IpAddrKind {
	V4,
	V6,
}

struct IpAddr {
	kind: IpAddrKind,
	address: String,
}

let home = IpAddr {
	kind: IpAddrKind::V4,
	address: String::from("127.0.0.1"),
};

let loopback = IpAddr {
	kind: IpAddrKind::V6,
	address: String::from("::1"),
};
```
- A more consise implementation here
```rust
enum IpAddr {
	V4(String),
	V6(String),
}

let home = IpAddr::V4(String::from("127.0.0.1"));

let loopback = IpAddr::V6(String::from("::1"));

enum IpAddr {
	V4(u8, u8, u8, u8),
	V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);

let loopback = IpAddr::V6(String::from("::1"));
```
- There is one more similarity between enums and structs: just as we’re able to define methods on structs using `impl`, we’re also able to define methods on enums. Here’s a method named `call` that we could define on our `Message` enum
```rust
impl Message {
	fn call(&self) {
		// method body would be defined here
	}
}

let m = Message::Write(String::from("hello"));
m.call();
```
#### The `Option` Enum and Its Advantages Over Null Values
Programming language design is often thought of in terms of which features you include, but the features you exclude are important too. **Rust doesn’t have the null feature that many other languages have.** _Null_ is a value that means there is no value there. In languages with null, variables can always be in one of two states: null or not-null.
- “[[PL/Null References---The Billion Dollar Mistake\|Null References---The Billion Dollar Mistake]]”, Tony Hoare
- 
The problem with null values is that if you try to use a null value as a not-null value, you’ll get an error of some kind. Because this null or not-null property is pervasive, it’s extremely easy to make this kind of error.

However, the concept that null is trying to express is still a useful one: a null is a value that is currently invalid or absent for some reason.

The problem isn’t really with the concept but with the particular implementation. As such, Rust does not have nulls, but it does have an enum that can encode the concept of a value being present or absent. This enum is `Option<T>`, and it is [defined by the standard library](https://doc.rust-lang.org/stable/std/option/enum.Option.html) as follows:

```rust
enum Option<T> {
    None,
    Some(T),
}

let some_number = Some(5);
let some_char = Some('e');
let absent_number: Option<i32> = None;
```
- Use [expect](https://doc.rust-lang.org/std/option/enum.Option.html#method.expect) or  [unwrap](https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap) can get the contained `Some` value, panic if equals `None`

### The `match` Control Flow Construct
```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
	match x {
		None => None,
		Some(i) => Some(i + 1),
	}
}

let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
let dice_roll = 9;

match dice_roll {
	3 => add_fancy_hat(),
	7 => remove_fancy_hat(),
	_ => reroll(),
}

fn add_fancy_hat() {}
fn remove_fancy_hat() {}
fn reroll() {}
```
### Concise Control Flow with `if let`
```rust
let mut count = 0;
match coin {
	Coin::Quarter(state) => println!("State quarter from {state:?}!"),
	_ => count += 1,
}

let mut count = 0;
if let Coin::Quarter(state) = coin {
	println!("State quarter from {state:?}!");
} else {
	count += 1;
}

```
### Reference
- [Do I need to have 64 bit Processor to use 64 bit data type](https://stackoverflow.com/questions/5530906/do-i-need-to-have-64-bit-processor-to-use-64-bit-data-type)
- [That’s Not Normal–the Performance of Odd Floats](https://randomascii.wordpress.com/2012/05/20/thats-not-normalthe-performance-of-odd-floats/)