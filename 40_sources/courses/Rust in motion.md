---
tags:
feature:
type:
author:
source:
---
# Rust in motion

The build manager and bulid tool for rust is **Cargo**

To create a new project:
`cargo new --bin project_name`
To create a new Library:
`cargo new --lib lib_name`

To compile (by default in debug directory)
`cargo build`
To compile for production
`cargo build --release` 

To run a program simply execute the 'executable' ./folder/filename

To compile and run the program
`cargo run`

To read documentation
`rustup doc`

## Variables
- all variables have a type
	- the type is after `:`
- variables can or can't be changed
	- by default variables cannot vary (immutability by default)
	- to allow mutability `let mut x = 5`
to declare a variable `let x = 5;`

## Data Types
#### Boolean
- **bool**: boolean
#### Integrals 
integrals (no decimals) can be signed or unsigned

| signed |              | unsigned |         | java  |
| ------ | ------------ | -------- | ------- | ----- |
| i8     | -128 / + 127 | u8       | 0 / 255 | byte  |
| i16    |              | u16      |         | short |
| i32    |              | u32      |         | int   |
| i64    |              | u64      |         | long  |
- `isize`and `usize` are architecture dependent (on 32 bits machine takes 32 bits on 64 bits machine takes 64 bits). This is the size of a pointer on each of this architecture
  **isize** and **usize** are used for indexing into collections or counting items
```rust
let a = [100, 200, 300];
let b = a[0] //type usize
```

default type for number is `i32` example `let x = 5 //type i32`

#### Floating point numbers
- numbers with decimal point
- `f32` and `f64`
- default `f64`
#### Characters
- `char` 
- unicode scalar value
- not only ASCII
- characters are in single quote `let c = 'a'`
#### Tuple
- group multiple values into one type
- values don't have to be the same type
- `let tup = (1, 'c', true)`
- to take a value out of a tuple we use indexing 
```rust
let tup = (1, 'c', true);
let first = tup.0;
let second = tup.1
```

destructuring allows to assign values to variables from a tuple
```rust
let tup = (1, 'c', true);
let (x, y, z) = tup
```
#### Arrays
- group multiple values into one value
- MUST have the same type
- `let a = [0.0, 3.14. -8.7928]`
- access the values by index `let second = a[1];`
- arrays have a fix length when initialised
- if need to change size use `vec`
#### Slice
- reference to a contiguous subset of data in another data structure
```rust
let a = [100, 200, 300];
let b = &a[0..1]; //0 inclusive 1 exclusive = 100
```


## Conditionals 
```rust
let a = true;
if a {
	println!("a is true)
}
```

## Functions
```rust
fn name(param1: type, ....) -> return_type {
	...body...
}
```

## Control Flow
