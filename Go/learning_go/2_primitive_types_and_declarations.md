# Primitive Types and Declarations

## Built-in Types
Go has many of the same built-in types as other languages: booleans, integers, floats, and strings. Using these types idiomatically is sometimes a challenge for developers coming from another language. In the following section we'll look at these types and see how they work best in Go.

### The Zero Value
Like most modern languages go assigns a default *zero value* to any variable that is declared but not assigned a value. Having an explicit zero value makes code clearer and removes a source of bugs found in C and C++ programs. 

### Literals
A *literal* in Go refers to writing out a number, character, or string. There are four common kinds of literals that you'll find in Go programs.

- *Integer literals*: A sequence of numbers, usually base 10 but prefixes can be used to indicate other bases(`0b` for binary, `0o` for octal, `0x` for hexadecimal).
- *Floating point literals*: Have decimal points to indicate the fraction portion of the value. Can also have an exponent specified like the following `6.03e23`.
- *Rune literals*: Represent characters and are surrounded by single quotes. Single and double quotes are not interchangeable in Go. Rune literals can be written as single Unicode characters(`'a'`), 32-bit Unicode numbers(`'\U00000061'`), and few other formats.
- *String literals*: There are two different types of string literals, *interpreted string literals* and *raw string literals*. Most of the time you will use double quotes to create an interpreted string literal(`"Greetings"`). These contain zero or more rune literals in any of the forms allowed.

Literals in Go are untyped, meaning they can interact with any variable that's compatible with the literal. For example, you can use an integer literal in floating point expressions or even assign an integer literal to a floating point variable. When we go over user defined types in [section 7](7_types_methods_interfaces.md), we'll see that we can even use literals with user-defined types based on primitive types. That being said you can't assign a literal string to a variable with a numeric type or a literal number to a string variable, nor can you assign a float literal to an int. These are all flagged by the compiler as errors.

There are situations in Go where the type isn't explicitly declared. In those cases Go uses the *default type* for a literal. 

### Booleans
The `bool` type represents Boolean variables. Variables of `bool` can have one of two values: `true` or `false`. The zero value for a `bool` is `false`.

```Go
var flag bool // no value assigned, set to false
var isAwesome = true
```

### Numeric Types
Go has a large number of numeric types that are grouped into three categories: integer, floating point, and complex. Some types are used frequently while others are more esoteric.

#### Integer Types
| Type   | Value Range                                 |
|--------|---------------------------------------------|
| int8   | -128 to 127                                 |
| int16  | –32768 to 32767                             |
| int32  | –2147483648 to 2147483647                   |
| int64  | –9223372036854775808 to 9223372036854775807 |
| uint8  | 0 to 255                                    |
| uint16 | 0 to 65536                                  |
| uint32 | 0 to 4294967295                             |
| uint64 | 0 to 18446744073709551615                   |

Go provides both signed and unsigned integers in a variety of sizes, from one to four bytes. And the zero value for all of the integer types is 0.

Go does have some special names for integer types. A `byte` is an alias for `uint8` and it is legal to compare, or perform mathematical operations between a `byte` and a `uint8`. However, you rarely see `uint8`, just use `byte` instead.

The second special name is `int`. On a 32-bit CPU, `int` is a 32-bit signed integer, like `int32`. On most 64-bit CPUs, `int` is a 64-bit signed integer, just like `int64`. Because `int` isn't consistent from platform to platform it is a compile time error to assign, compare, or perform mathematical operations between an `int` and an `int32` or `int64` without a type conversion. Integer literals default to being of `int` type.

The third special name is `uint`, it follows the same rules as `int` only it is unsigned.

Go provides more integer types than some other languages, choosing which type to use comes down to the following three rules:

- If you are working with a binary file format or network protocol that has an integer of a specific size or sign, use the corresponding integer type.
- If you are writing a library function that should work with any integer type, write a pair of functions, one with `int64` for the parameters and variables and the other with `uint64`.(More on functions and their parameters in [section 5](5_functions.md))
- In all other cases just use `int`.

#### Floating Point Types
There are two floating point types in Go.

| Type    | Largest Absolute Value                          | Smallest (nonzero) absolute value              |
|---------|-------------------------------------------------|------------------------------------------------|
| float32 | 3.40282346638528859811704183484516925440e+38    | 1.401298464324817070923729583289916131280e-45  |
| float64 | 1.797693134862315708145274237317043567981e +308 | 4.940656458412465441765687928682213723651e-324 |

Picking which floating point type to use is straightforward: unless you have to be compatible with an existing format, use `float64`. 

The bigger question is whether you should be using floating point numbers at all. In most cases, the answer is no. Just like other languages, Go floating point numbers have a huge range, but they cannot store every value in that range, they store the nearest approximation. Because floats aren't exact, they can only be used in situations where inexact values are acceptable or the rules of floating point are well understood. That limits them to things like graphics and scientific operations.

#### Complex Types
You probably won't use these very much. Go defines two complex number types `complex64` uses `float32` values to represent the real and imaginary part, and `complex128` uses `float64`. Both are declared with the `complex` built-in function.

### A Taste of Strings and Runes
Like most modern languages Go includes a built-in type for strings. The zero value for a string is the empty string and Go supports Unicode. Like integers and floats, string are compared for equality using the `==`, difference with `!=`, or ordering with `>`,`>=`,`<`, or `<=`. They are concatenated by using the `+` operator.

Strings in Go are immutable, you can reassign the value of a string variable, but you cannot change the value of the string that is assigned to it.

Go also has a type that represents a single code point. The *rune* type is an alias for thee `int32` type, just like `byte` is an alias for `uint8`. 

We'll cover more about strings, including implementation details, relationships with bytes and runes, as well as advanced features in the next [section](3_composite_types.md)

