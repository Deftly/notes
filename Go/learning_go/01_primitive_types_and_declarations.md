# Primitive Types and Declarations

<!--toc:start-->
- [Primitive Types and Declarations](#primitive-types-and-declarations)
  - [Built-in Types](#built-in-types)
    - [The Zero Value](#the-zero-value)
    - [Literals](#literals)
    - [Booleans](#booleans)
    - [Numeric Types](#numeric-types)
      - [Integer types](#integer-types)
      - [The special integer types](#the-special-integer-types)
      - [Integer operators](#integer-operators)
      - [Floating point types](#floating-point-types)
      - [Complex types(you're probably not going to use these)](#complex-typesyoure-probably-not-going-to-use-these)
    - [A Taste of Strings and Runes](#a-taste-of-strings-and-runes)
    - [Explicit Type Conversion](#explicit-type-conversion)
  - [`var` Versus `:=`](#var-versus)
  - [Using `const`](#using-const)
  - [Typed and Untyped Constants](#typed-and-untyped-constants)
  - [Unused Variables](#unused-variables)
  - [Naming Variables and Constants](#naming-variables-and-constants)
  - [Wrapping Up](#wrapping-up)
<!--toc:end-->

## Built-in Types
Go has many of the same built-in types as other languages: booleans, integers, floats, and strings. This sections will cover these types and how to best use them. Before that we'll cover some concepts that apply to all types.

### The Zero Value
Go assigns a default *zero value* to any variable that is declared but not assigned a value. This makes code clearer and removes a source of bugs found in C and C++.

### Literals
A literal in Go refers to writing out a number, character, or string. There are four common kinds of literals that you'll find in Go programs

*Integer literals* are sequences of numbers, usually base ten, but different prefixes can be used to indicate other bases: `0b` for binary, `0o` for octal, and `0x` for hexadecimal.

*Floating point literals* have decimal points to indicate the fractional portion of a value. They can also have an exponent specified with the letter e and a positive or negative number(such as `6.03e23`).

*Rune literals* represent characters and are surrounded by single quotes. They can be written as single Unicode characters(`'a'`), 8-bit octal numbers(`'\141'`), 16-bit hexadecimal numbers(`'\u0061'`), or 32-bit Unicode numbers(`'\U00000061'`).

There are two different ways to indicate *string literals*. You'll mostly use double quotes to create an *interpreted string literal* which will contain zero or more rune literals in any of the allowed forms. The only characters that cannot appear are unescaped backslashes, unescaped newlines, and unescaped double quotes. Here is an example: `"Greetings and \n\"Salutations\""`. 

The other type of string literal is a *raw string literal*. These are delimited using backquotes(`) and can contain any literal character except a backquote:
```go
`Greetings and 
"Saluations"`
```
Go doesn't allow operations between two integer variables if they are declared to be of different sizes. However, you can use an integer literal in a floating point expression or even assign an integer literal to a floating point variable. This is because literals in Go are untyped, meaning they can interact with any variable that's compatible with the literal. We can even use literals with user-defined types based on primitive types.

Being untyped doesn't mean you can assign a literal string to variable with a numeric type, nor can you assign a literal whose value overflows the specified variable's type. These will be flagged by the compiler as errors.

There are situations in Go where the type isn't explicitly declared. In those cases, Go uses the *default type* of the literal as the type.

### Booleans
The `bool` type represents Boolean variables which can have one of two values: `true` or `false`. The zero value for a `bool` is `false`:
```go
var flag bool // no value assigned, set to false
var isAwesome = true
```

### Numeric Types
Go has 12 different numeric types(and a few special names/aliases) that are grouped into 3 categories(integer types, floating point types, and complex types).

#### Integer types
Go provides both signed and unsigned integers in a variety of sizes:
| Type   | Value range    |
|--------------- | --------------- |
| int8 | -128 to 127 |
| int16 | -32768 to 32767 |
| int32 | -2147483648 to 2147483647 |
| int64 | -9223372036854775808 to -9223372036854775807 |
| uint8 | 0 to 255 |
| uint16 | 0 to 65536 |
| uint32 | 0 to 4294967295 |
| uint64 | 0 to 18446744073709551615 |

The zero value for all integer types is 0.

#### The special integer types
A `byte` is an alias for `uint8`. It is legal to assign, compare, or perform mathematical operations between a `byte` and a `uint8`. You will rarely see `uint8` used, just use `byte` instead.

The second name is `int`. On a 32-bit CPU, `int` is a 32-bit signed integer like `int32`. On most 64-bit CPUs, `int` is a 64-bit signed integer like `int64`. Because `int` isn't consistent from platform to platform it is a compile time error to assign, compare, or perform mathematical operations between an `int` and an `int32` or `int64` without a type conversion. Integer literals default to being of type `int`.

The third special name `uint` which follows the same rules as `int`, only it is unsigned.

The two other special names for integer types are `rune` and `uintptr`.

> **_NOTE:_** Unless you need to be explicit about the size of an integer for performance or integration purposes, use the `int` type.

#### Integer operators
Go integers support the usual arithmetic operators: `+`, `-`, `*`, `/`, and `%` for modulus. The result of integer division is an integer, to get a floating point result you need to use type conversion to make your integers into floating point numbers.

Go also has bit-manipulation operators for integers. You can bit shift left and right with `<<` and `>>`, or do bit masks with `&`(logical AND), `|`(logical OR), `^`(logical XOR), and `&^`(logical AND NOT).

#### Floating point types
There are two floating point types in Go, `float32` and `float64`. They have a zero value of 0 and function similar to floating point numbers in other languages with a large range and limited precision. Floating point literals have a default type of `float64` and you should always use `float64` unless you need to be compatible with an existing format calls for `float32`.

Because floats aren't exact they can only be used in situations where inexact values are acceptable or the rules of floating point are well understood, limiting their use to things like graphics and scientific operations. Never use them to represent money or any other value that needs an exact decimal representation.

You can use all the standard mathematical and comparison operators with floats, except `%`. Floating point division has a couple interesting properties. Dividing a nonzero floating point variable by 0 returns `+Inf` or `-Inf`(positive or negative infinity) depending on the sign of the number. Dividing a floating point variable set to 0 by 0 returns `NaN`(Not a Number).

While Go allows you to use `==` and `!=` to compare floats you shouldn't due to the inexact nature of floats. Instead define a maximum allowed variance and see if the difference between two floats is less than that.

#### Complex types(you're probably not going to use these)
While Go does have complex numbers as a built-in type it is not a popular language for numerical computation. This is likely because features like matrix support are not part of the language and libraries have to use inefficient replacements. There has been discussion about removing complex numbers from a future version of Go but it's easier to just ignore the feature.

### A Taste of Strings and Runes
Like most languages Go includes strings as a built-in type and the zero value for a string is an empty string. You can put any Unicode character into a string. Like integers and floats they can be compared for equality using `==`, difference with `!=`, or ordering with `>`, `>=`, `<`, or `<=`.

Strings in Go are immutable, you can reassign the value of a string variable, but you cannot change the value of the string that is assigned to it.

Go also has a type that represents a single code point. The `rune` type is an alias for `int32` just like `byte` is an alias for `uint8`. As you could probably guess, a rune literals default type is a `rune`, and a string literal's default type is a `string`.

> **_NOTE:_** If you are referring to a character use the `rune` type not the `int32` type. They might be the same to the compiler but you want to use the type that clarifies the intent of your code.

### Explicit Type Conversion
As a language that values clarity of intent and readability, Go doesn't allow automatic type promotion between variables. You must use a *type conversion* when variable types do not match. Even different sized integers and floats must be converted to the same type to interact.
```go
var x int = 10
var y float64 = 30.2
var z float64 = float64(x) + y
var d int = x + int(y)
```
Since all type conversions in Go are explicit, you cannot treat another Go type as a boolean. Many languages allow a nonzero number or nonempty string to be interpreted as a boolean `true`. In Go *no other type can be converted to a `bool`, implicitly or explicitly*. To convert from another data type to a boolean, you must use one of the comparison operators.

## `var` Versus `:=`
The most verbose way to declare a variable in Go uses the `var` keyword, and explicit type, and an assignment:
```go
var x int = 10
```
If the value on right side of the `=` is the expected type of your variable you can leave off the type on the left side of the `=`.
```go
var x = 10
```
Conversely, if you want to declare a variable and assign it the zero value, you can keep the type and drop the `=`.
```go
var x int
```
You can also declare multiple variables at once:
```go
var x, y int = 10, 20 // multiple assignment
var a, b int // assigned zero values of the same type
var c, d = 10, "hello" // different types
var (
    x1 int
    y1     = 20
    z1 int = 30
    d1, e1 = 40, "hello"
    f1, g1 string
)
```
Go also supports a short declaration format. When you are within a function, you can use the `:=` operator to replace a `var` declaration that uses type inference.
```go
x := 10 // equivalent to var x = 10
```
Like `var` you can declare multiple variables at once using `:=`. An additional trick you can do with `:=` that you can't do with `var` is assign values to existing variables too. As long as there is one new variable on the left hand side of the `:=` then any of the other variables can already exist:
```go
x := 10 
x, y := 30, "hello"
```
There are some situations where you want to avoid using the short declaration format:
- When initializing a variable to its zero value use `var x int`, this makes it clear the zero value is intended.
- When assigning an untyped constant or literal to a variable and the default type for the constant or literal isn't the type you want for the variable use the long `var` format(`var x byte = 20`)

> **_NOTE:_** Avoid declaring variables outside of functions because they complicate data flow analysis.

## Using `const`
In Go you declare a value to be immutable using the `const` keyword:
```go 
const x int64 = 10

const (
  idKey   = "id"
  nameKey = "name"
)

const z = 20 * 10

func main() {
  const y = "hello"
  fmt.Println(x)
  fmt.Println(y)
  x = x + 1   // compilation will fail with an error: cannot assign to x
  y = "bye"   // compilation will fail with an error: cannot assign to y
}
```
Constants in Go are a way to give names to literals. They can only hold values that the compiler can figure out at compile time. This means they can be assigned:
- Numeric literals
- `true` and `false`
- String literals
- Rune literals
- The built-in functions `complex`, `real`, `len`, and `cap`
- Expressions that consist of operators and the preceding values

Go doesn't provide a way to specify that a value calculated at runtime is immutable. So there are no immutable arrays, slices, maps, or structs, and there is not way to declare that a field in a struct is immutable. This isn't as limiting as it sounds. Within a function it is obvious if a variable is being modified so immutability is less important. In a later section we'll see how Go prevents modifications to variables that are passed as parameters to functions.

> **_NOTE:_** Constant in Go are a way to give names to literals. There is *no* way in Go to declare that a variable is immutable.

## Typed and Untyped Constants
Constants can be typed or untyped. An untyped constant works exactly like a literal. It has no type of its own but does have a default value that is used when no other type can be inferred. A typed constant can only be directly assigned to a variable of that type.

In general leaving constants untyped gives you more flexibility. There are some situations where you want a constant to enforce a type like when we create enumerations with `iota` which we will discuss later.
```go
const x = 10
var y int = x
var z float64 = x
var d byte = x

const typedX int = 10
z = typedX // results in an error: cannot use typedX (type int) as a float64 in assignment 
```
## Unused Variables
To make it easier for large teams to collaborate on programs Go has some unique rules, one of which is that *every declared local variable must be read*. It is a compile time error to declare a local variable and to not read its value.

> **_NOTE:_** The Go compiler does allow you to create unread constants with `const`. This is because constants in Go are calculated at compile time and cannot have any side effects. If a constant isn't used, it is simply not included in the compiled binary.

## Naming Variables and Constants
Go requires identifier names to start with a letter or underscore, and the name can contain numbers, underscores, and letters. Although underscore is a valid character in a variable name it is rarely used because idiomatic Go doesn't use snake case(names like `index_counter` or `number_tries`). Instead, Go developers use camel case(names like `indexCounter` and `numberTries`) when an identifier name consists of multiple words.

In many languages, constants are always written in all uppercase letters, with words separated by underscores(names like `INDEX_COUNTER`). Go doesn't follow this pattern because Go uses the first letter in the name of a package-level variable to determine if the item is accessible outside the package.

## Wrapping Up
This section covered how to use built-in types, declare variables and work with assignments and operators. The next section will go over composite types in Go: arrays, slices, maps, and structs. We'll also take another look at strings and runes and learn about encodings.
