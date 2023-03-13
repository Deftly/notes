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
jvar flag bool // no value assigned, set to false
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

## Explicit Type Conversion
Go doesn't allow automatic type promotion between variables in order to maintain clear intent and readability. You must use *type conversion* when variable types do not match. Even different sized integers and floats must be converted to the same type to interact. This makes it clear exactly what type you want without having to memorize any type conversion rules.

```Go
var x int = 10
var y float64 = 30.2
var z float64 = float64(x) + y
var d int = x + int(y)
```

Since all type conversions in Go are explicit, you cannot treat another Go type as a boolean. In many languages, a nonzero number or nonempty string can be interpreted as a boolean `true`. Just like automatic type promotion, the rules for "truthy" values vary from language to language and can be confusing. So unsurprisingly, Go doesn't allow truthiness. In fact, *no other type can be converted to a bool, implicitly or explicitly*. If you want to convert from another data type to boolean, you must use one of the comparison operators. 

## var Versus :=
The most verbose way to declare a variable in Go uses the `var` keyword, an explicit type, and an assignment:

```Go
var x int = 10
```

If the default type of the literal is the expected type of your variable you leave off the type on the left side of the `=`. Conversely, if you want to declare a variable and assign it the zero value, you can keep the type and drop the `=`. Here are a few other ways you can declare variables with `var`:

```Go
var x int
var x, y int = 10, 20
var x, y int
var x, y = 10, "hello"
var (
    x    int
    y        = 20
    z    int = 30
    d, e     = 40, "hello"
    f, g string 
)
```

Go also supports a short declaration format. When you are within a function, you can use the `:=` operator to replace a `var` declaration that uses type inference. The following two statements do exactly the same thing:

```Go
var x = 10
x := 10
```

Like `var`, you can declare multiple variables at once using `:=`:

```Go
var x, y = 10, "hello"
x, y := 10, "hello"
```

The `:=` can do one thing you cannot do with `var`, it allows you to assign values to existing variables, too. So long as there is one new variable on the left hand side of the `:=`, then any of the other variables can already exist:

```Go
x := 10
x, y := 30, "hello"
```

The one limitation on `:=` is that for declaring package level variables you must use `var` because `:=` is not legal outside of functions.

When choosing between `var` and `:=` always choose what makes your intent clearest. The most common declaration style within functions is `:=`, though there are some situations within functions where you should avoid `:=`:
- When initializing a variable to its zero value, use `var`. This makes it clear that the zero value is intended.
- When assigning an untyped constant or a literal to a variable and the default type for the constant or literal isn't the type you want for the variable.
- Because `:=` allows you to assign to both new and existing variables, it sometimes creates new variables when you think you are reusing existing ones([Shadowing variables]()). In those situations, explicitly declare all of your new variables with `var` to make it clear which variables are new, and then use the assignment operator to assign values to both new and old variables.

You should generally avoid declaring variables outside of functions because they complicate data flow analysis and can result in subtle bugs that are hard to understand and fix.

## Using const
Many languages have a way to declare a value is immutable. In Go, this is done with the `const` keyword. At first glance, it seems to work exactly like other languages. 

```Go
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

  x = x + 1
  y = "bye"

  fmt.Println(x)
  fmt.Println(y)
}
```

If you try to run this code, compilations fails with the following error messages:
```bash
./const/go:20:4: cannot assign to x
./const.go:21:4: cannot assign to y
```

Constants in Go are a way to give names to literals. They can only hold values that the compiler can figure out at compile time. This means they can be assigned:
- Numeric literals
- `true` and `false`
- Strings
- Runes
- The built-in functions `complex`, `real`, `imag`, `len`, and `cap`
- Expressions that consist of operators and the preceding values

Go doesn't provide a way to specify that a value calculated at runtime is immutable. There are no immutable arrays, slices, maps, or structs, and there's no way to declare that a field in a struct is immutable. This isn't as limiting as it sounds. Within a function, it is clear if a variable is being modified, so immutability is less important. In [section 5(Go is Call by Value)](5_functions.md), we'll see how Go prevents modifications to variables that are passed as parameters to functions.

## Typed and Untyped Constants
Constants can be typed or untyped. An untyped constant works exactly like a literal, it has no type of its own but does have a default type that is used when no other type can be inferred. A typed constant can only be directly assigned to a variable of that type. 

In general, leaving a constant untyped gives you more flexibility, however there are some situations where you want a constant to enforce a type. We'll use typed constants when we look at creating enumerations with `iota` in [section 7(iota Is for Enumerations-Sometimes)](7_types_methods_interfaces.md).

```Go
const x = 10 // Untyped constant declaration

// All of the following assingments are legal:
var y int = x
var z float64 = x
var d byte = x

const typedX int = 10 // Typed constant declaration, this constant can only be assigned directly to an int
```

## Unused Variables
Another Go requirement is that *every declared local variable must be read*. It is a compile-time error to declare a local variable and to not read its value.

Perhaps surprisingly, the Go compiler allows you to create unread constants with `const`. This is because constants in Go are calculated at compile time and cannot have any side effects. This makes them easy to eliminate: if a constant isn't used, it is simply not included in the compiled binary.

## Wrapping Up
This section covers how to use the built-in types, declaring variables, and assignments. In the next [section](3_composite_types.md) we are going to go over composite types in GO: arrays, slices, maps, and structs. We'll also take another look at strings and runes and learn about encodings.
