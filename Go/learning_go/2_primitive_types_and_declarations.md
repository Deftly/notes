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

