# Primitive Types and Declarations
In this section we'll begin looking at some of Go's language features and how to best use them. When trying to figure out what "best" means the overriding principle is to make our intentions clear. 

This section will cover primitive types and variables. While every programmer will have experience with these concepts Go does some things differently and there are subtle difference between Go and other languages.

## Built-in Types
Go has many of the same built-in types as other languages: booleans, integers, floats, and strings. We'll cover each of these types and how to best use them in Go, but before that we'll review some of the concepts that apply to all types.

### The Zero Value
Like most modern languages, Go assigns a default *zero value* to any variable that is declared but not assigned a value. As we cover each type we will also cover the zero value for that type. 

### Literals
A literal in Go refers to writing out a number, character, or string. 

*Integer literals* are a sequence of numbers, normally base ten, but different prefixes can be used to indicate other bases: `0b` for binary, `0o` for octal, or `0x` for hexadecimal. 

*Floating point literals* have decimal points to indicate the fractional portion of the value. They can also have an exponent specified with the letter e and a positive or negative number(such as `6.03e23`). They can also be written in hexadecimal by using the `0x` prefix and the letter `p` for indicating any exponent. 

*Rune literals* represent characters and are surrounded by single quotes. Unlike many other languages, in Go single quotes and double quotes are not interchangeable. Rune literals can be written as single Unicode characters(`'a'`), 8-bit octal numbers(`'\141'`), 16-bit hexadecimal numbers(`'\u0061'`), or 32-bit Unicode numbers(`'U00000061'`). There are also several backslash escaped rune literals, the most use being newline(`'\n'`), tab(`'\t'`), single quote(`'\''`), double quote(`'\"'`), and backslash(`'\\'`). 

Practically speaking, use base ten to represent your number literals and, unless the context makes your code clearer, try to avoid using any of hexadecimal escapes for rune literals.

There are two different ways to indicate *string literals*. Most of the time, you should use double quotes to create an *interpreted string literal*(e.g `"Greetings and Salutations"`). These contain zero or more rune literals in any of the forms allowed.

*Raw string literals* are delimited with backquotes(`) and can contain any literal character except a backquote. When using a raw string literal we can write multiline greetings like so.

```go
`Greetings and
"Salutations"`
```

As we'll see shortly you can't add two integer variables together if they are declared to be of different sizes. However, Go lets you use an integer literal in floating point expressions and even assign an integer literal to a floating point variable. This is because literals in Go are untyped, they can interact with any variable that's compatible with the literal. Later we'll see that we can even use literals with user-defined types based on primitive types.

There are situations in Go where the type isn't explicitly declared. In those cases, Go uses the *default type* for a literal. We'll mention the default type for literals as we look at the different built-in types.

### Booleans
The `bool` type represents Boolean variables. Variables of `bool` type can have one of two values: `true` or `false`. The zero value for a `bool` is `false`:

```go
var flag bool // no value assigned, set to false
var isAwesome = true
```

### Numeric Types
Go has 12 numeric types that are grouped into three categories, some of which are frequently used others are more esoteric. We'll start with integer types then move to floating point types and finally complex types.

#### Integer types
Go provides both singed and unsigned integers in a variety of sizes, from one to four bytes:

| Type name   | Value range    |
|--------------- | --------------- |
| int8   | -128 to 127   |
| int16 | -32768 to 32767 |
| int32   | -2147483648 to 2147483647  |
| in64   | -9223372036854775808 to  9223372036854775807  |
| uint8 | 0 to 255 |
| uint16 | 0 to 65536 |
| uint32 | 0 to 4294967295 |
| uint64 | 0 to 18446744073709551615 | 

The zero value for all of the integer types is 0.

#### The special integer types
Go does have some special names for integer types. A `byte` is an alias for `uint8` and it is legal to assign, compare, or perform mathematical operations between a `byte` and a `uint8`. However, you rarely see `uint8` used in Go code, just call it a `byte`.

The second special name is `int`. On a 32-bit CPU, `int` is a 32-bit signed integer like `int32`. On most 64-bit CPUs, `int` is a 64-bit signed integer like `int64`. Because it isn't consistent from platform to platform it is a compile-time error to perform mathematical operations between an `int` and an `int32` or `int64` without a type conversion. Integer literals default to being of `int` type.

The third special name is `uint`. It follows the same rules as `int`, only it is unsigned.

The two other special names for integer types are `rune` and `uintptr`. We'll seen rune literals previously and will discuss the `rune` type later in this section. `uintptr` will be covered in [section 14](./14_here_there_be_dragons:reflect_unsafe_and_cgo.md).

#### Choosing which integer to use
There are three simple rules when choosing which integer type to use:
- If you are working with a binary file format or network protocol that has an integer of a specific size or sign, use the corresponding integer type.
- If you are writing a library function that should work with any integer type, write a pair of functions, one with `int64` for the parameters and variables and the other with `uint64`.(More about functions and parameters in [section 5](./5_functions.md))
- In all other cases, just use `int`.

> Unless you *need* to be explicit about the size or sign of an integer for performance or integration purposes, use `int`. Using other types should be considered a premature optimization until proven otherwise. 

#### Integer operators
Go integers support the usual arithmetic operators: `+`,`-`,`*`,`/`, with `%` for modulus. The result of an integer division is an integer. To get a floating point result, you need a type conversion. 

> Integer division in Go follows truncation toward zero, read the go spec's section [arithmetic operators](https://go.dev/ref/spec#Arithmetic_operators) for full details.

You can combine any of the arithmetic operators with `=` to modify a variable: `+=`, `-=`, `*=`, `/=`, and `%=`. You can compare integers with `==`, `!=`, `>`, `>=`, `<`, and `<=`.

Go also has bit-manipulation operators for integers. You can bit shift left and right with `<<` and `>>`, or dobit masks with `&`(logical AND), `|`(logical OR), `^`(logical XOR), and `&^`(logical AND NOT). Just like arithmetic operators, you can also combine all of the logical operators with `=` to modify a variable: `&=`, `|=`, `^=`, `&^=`, `<<=`, `>>=`. 

#### Floating point types
There are two floating point types in Go:

| Type name | Largest absolute value  | Smallest(nonzero) absolute value |
|---------------- | --------------- | --------------- |
| float32 | 3.40282346638528859811704183484516925440e+38 | 1.401298464324817070923729583289916131280e-45|
| float64 | 1.797693134862315708145274237317043567981e+308 | 4.940656458412465441765687928682213723651e-324 |

Like integer types, the zero value for the floating point types is 0.

Go uses the IEEE 754 specification for floats, giving a large range and limited precision. Choosing which floating point type to use is simple: unless you need to be compatible with an existing format, use `float64`. Floating point literals have a default type of `float64`, so always using `float64` is the simplest option. This also reduces floating point accuracy issues since `float32` only has six-seven decimal digits of precision. You shouldn't worry about the difference in memory size unless you have used a profiler to determine that it is a significant source of problems.(Testing and profiling will be covered in [section 13](./13_writing_tests.md))

In most cases you don't want to use floating point numbers at all. This is because floats aren't exact and so can only be used in situations where inexact values are acceptable or the rules of floating point are well understood(graphics and scientific operations)

> A floating point number cannot represent a decimal value exactly. Never use them to represent money or any other value that must have an exact decimal representation!

You can use all the standard mathematical and comparison operators with floats, except `%`. Dividing a nonzero floating point variable by 0 returns `+Inf` or `-Inf` depending on the sign of the number. Dividing a floating point variable set to 0 by 0 returns `NaN`(Not a number).

While Go allows the use of `==` and `!=` to compare floats you shouldn't. Due to the inexact nature of floats, two floating point values might not be equal when you think they should be. Instead, define a maximum allowed variance and see if the difference(epsilon) is less than that.

#### Complex types(you're probably not going to use these)
Go has first-class support for complex numbers. If you don't know what complex numbers are feel free to skip ahead.

Go defines two complex number types. `complex64` uses `float32` values to represent the real and imagine part and `complex128` uses `float64` values. Both are declared with the `complex` built-in function. Go uses a few rules to determine what the type of the function output is:
- If you use untyped constants or literals for both function parameters you will create an untyped complex literal, which has a default type of `complex128`.
- If both values passed into `complex` are of `float32` type, you'll create a `complex64`.
- If one value is `float32` and the other is an untyped constant or literal that can fit within a `float32`, you'll create a `complex64`
- Otherwise, you'll create a `complex128`.

All the standard arithmetic operators work on complex numbers, however, just like floats they have precision limits so it's best to use the epsilon technique for comparison. You can extract the real and imaginary portions of complex numbers with the `real` and `imag` built-in functions, respectively. The zero value for both types of complex numbers has 0 assigned to both the real and imaginary portions of the number. 

```go
func main() {
    x := complex(2.5, 3.1)
    y := complex(10.2 2)
    fmt.Println(x + y) // (12.7+5.1i)
    fmt.Println(x - y) // (-7.699999999999999+1.1i)
    fmt.Println(x * y) // (19.3+36.62i)
    fmt.Println(x / y) // (0.2934098482043688+0.24639022584228065i)
    fmt.Println(real(x)) // (2.5)
    fmt.Println(imag(x)) // (3.1)
    fmt.Println(cmplx.Abs(x)) // 3.982461550347975
}
```

You can see floating point imprecision on display here as well. And while GO has complex numbers as a built-in type Go is not a popular choice for numerical computing. This is because features like matrix support are not a part of the language and libraries have to use inefficient replacements, like slices of slices. 

### A Taste of Strings and Runes
Go includes strings as a built-in type, the zero value for a string is the empty string. Go supports Unicode, as we saw before you can put any Unicode character into a string. Like integers and floats, strings can be compared using `==`, `!=`, `>`, `>=`, `<`, or `<=`.

Strings in Go are immutable, you can reassign the value of a string variable but you cannot change the value of the string that is assigned to it. 

Go also has a type that represents a single code point. The *rune* type is an alias for `int32`. A rune literal's default type is a rune and a string literal's default type is a string. 

The [next section](./3_composite_types.md) will go through strings in more detail. It will cover some implementation details, relationships with bytes and runes, as well as advanced features and pitfalls.

### Explicit Type Conversion
Go doesn't allow automatic type promotion between variables, you must use a *type conversion* when variable types don't match. Even different sized integers and floats must be converted to the same type to interact, this make it clear exactly what type you want without having to memorize any type conversion rules.

```go
var x int = 10
var y float64 = 30.2 
var z float64 = float64(x) + y
var d int = x + int(y) 
```

> Type conversion are one of the place where Go choose to add verbosity in exchange for simplicity and clarity. Idiomatic GO values comprehensibility over conciseness.

## var Versus := 
The most verbose way to declare a variable in Go uses the `var` keyword, an explicit type, and an assignment:

```go 
var x int = 10
```

If the type on the right hand side of the `=` is the expected type of your variable, you can leave off the type from the left hand side of the `=`:

```go
var x = 10
```

Conversely, to declare a variable and assign it the zero value keep the type and drop the `=`:

```go 
var x int
```

You can also declare multiple variables at once with var:

```go
var x, y int = 10, 20
var a, b int
var c, d = 10, "hello"
```

The final way to use `var` is to create a *declaration list*:

```go 
var (
    x int
    y = 20
    z int = 30
    d, e = 40, "hello"
    f, g string
)
```

Go supports a short declaration format. When you are within a function, you can use the `:=` operator to replace a `var` declaration that uses type inference. The following statements are equivalent:

```go 
// var x = 10 
x := 10 
```

Like `var` you can declare multiple variables at once using `:=`:

```go 
// var x, y = 10, "hello"
x, y := 10, "hello"
```

One thing you can do with `:=` that you cannot do with `var` is assign values to existing variables too. As long as there is one new variable on the left hand side of the `:=` any of the other variables can already exist.

```go 
x := 10 
x, y := 30, "hello"
```

The limitation of `:=` is that it is not legal outside of a function. SO for package level variables you must use `var`.

When choosing which style to use always choose what makes your intent clearest. The most common declaration style within a function is `:=`, though there are a few situations within functions where you should avoid `:=`.
- When initializing a variable to its zero value, use `var x int`. This makes it clear that the zero value is intended. 
- When assigning an untyped constant or literal to a variable and the default type for the constant or literal isn't the type you want for the variable. Use the long `var` form with the type specified. While it is legal to use type conversion it is idiomatic to write `var x byte = 20`.

While `var` and `:=` allow you to declare multiple variables on the same line, only use this style when assigning multiple values returned from a function or the comma ok idiom(see [section 5](./5_functions.md).

You should rarely use package level variables, and such variables whose values change are a bad idea. It can be hard to track changes made to package level variables which makes it hard to understand the flow of data through your program. As a general rule, you should only declare variables in the package block that are effectively immutable. 

## Using `const`
Many languages have a way to declare a value immutable. In Go, this is done with the `const` keyword:

```go 
const x int64 = 10 

const (
    idKey = "id"
    nameKey = "name"
)

const z = 20 * 10 

func main() {
    const y = "hello"

    fmt.Println(x)
    fmt.Println(y)

    x = x + 1 
    y = "bye"

    fmt.Println(x)
    fmt.Println(y)
}
```

When this code is run, compilation will fail with the following error messages:

```shell
cannot assign to x 
cannot assign to y
```

At first glance, it seems to work exactly like other languages, but `const` in Go is very limited. Constant in Go are a way to give names to literals. They can only hold values that the compiler can figure out at compile time, meaning they can be assigned to the following:
- Numeric literals
- `true` and `false`
- Strings
- Runes
- `iota`
- The built-in functions `complex`, `real`, `imag`, `len`, and `cap`
- Expressions that consist of operators and the preceding values

Go doesn't provide a way to specify that a value calculated at runtime is immutable, meaning there are no immutable arrays, slices, maps, structs, or struct fields. This isn't as limiting as it sounds. Within a function it is clear if a variable is being modified and in later sections we'll see how Go prevents modifications to variables that are passed as parameters to functions.

> Constants in Go are a way to give names to literals. There is ***no*** way in Go to declare that a variable is immutable.
