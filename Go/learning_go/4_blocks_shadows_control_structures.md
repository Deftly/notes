# Blocks, Shadows, and Control Structures
In This section we'll start by explaining blocks and how they control when an identifier is available. Then we'll look at Go's control structures: `if`, `for`, and `switch`. Finally, we will talk about `goto` and the one situation when you should use it.

## Blocks
Go lets you declare variables in lots of places. Outside of functions, as parameters to functions, and as local variables within functions.

Each place where a declaration occurs is called a *block*. Variables, constants, types, and functions declarations outside of any functions are placed in the *package* block. We've used `import` statements in our programs to gain access to functions(will talk about them in more detail in [section 9](./9_modules_packages_imports.md)). They define names for other packages that are valid for the file that contains the `import` statement. These names are in the *file* block. All variables at the top level of a function(including the parameters to a function) are in a block. Within a function, every set of braces(`{}`) defines another block.

You can access an identifier defined in any outer block from within any inner block. 

### Shadowing Variables
What happens when you have a declaration with the same name as an identifier in a containing block? Doing so means you *shadow* the identifier created in the outer block.

```go 
func main() {
    x := 10
    if x > 5 {
        fmt.Println(x) // 10
        x := 5 
        fmt.Println(x) // 5
    }
    fmt.Println(x) // 5
}
```

A shadowing variable is a variable that has the same name as a variable in a containing block. For as long as the shadowing variable exists, you cannot access a shadowed variable.

You also need to be careful to ensure that you don't shadow a package import. Let's see what happens when wee declare a variable called `fmt` within our `main` function:

```go 
func main() {
    x := 10 
    fmt.Println(x)
    fmt := "oops"
    fmt.Println(fmt)
}
```

Running this code will give us the following error:

```shell
fmt.Println undefined (type string has no field or method Println)
```

Notice the problem here isn't that we named our variable `fmt`, it's that we tried to access something that the local variable `fmt` didn't have. Once the local variable `fmt` is declared, it shadows the package named `fmt` in the file block, making it impossible to use the `fmt` package for the rest of the `main` function. 

### The Universe Block
There's actually one more block that is a little weird: The universe block. Go is a small language with only 25 keywords, what's interesting is that built-in types(like `int` and `string`), constants(like `true` and `false`), and functions(like `make` or `close`) aren't included in that list. Neither is `nil`. So where are they?

Rather than make them keywords, Go considers these *predeclared identifiers* and defines them in the universe block, which is the block that contains all other blocks.

Because these names are declared in the universe block, it means they can be shadowed in other scopes likes so:

```go 
fmt.Println(true) // true
true := 10
fmt.Println(true) // 10
```

**You must be very careful to never redefine any of the identifiers in the universe block**. Accidentally doing so will cause some very strange behavior. If you're lucky, you'll get compilation failures. If not, it can be difficult to track down the source of your problems.

## `if`
The `if` in Go is very similar to `if` statements in most languages.

```go 
n := rand.Intn(10)
if n == 0 {
    fmt.Println("That's too low")
} else if n > 5 {
    fmt.Println("That's too big:", n)
} else {
    fmt.Println("That's a good number:", n)
}
```

The most visible difference between `if` statements in Go and other languages is that you don't put parenthesis around the condition. But there's another feature that Go adds to `if` statements to better manage variables. 

Go adds the ability to declare variables that are scoped to the condition and to both the `if` and `else` blocks:

```go 
if n := rand.Intn(10); n == 0 {
    fmt.Println("That's too low")
} else if n > 5 {
    fmt.Println("That's too big:", n)
} else {
    fmt.Println("That's a good number:", n)
}
```

Having this special scope is very handy, it lets you create variables that are only available where they are needed. 

> Technically you put any *simple statement* before the comparison in an `if` statement. This includes function calls that don't return a value or assigning a new value to an existing variable. But don't do this. Only use this feature to define new variables that are scoped to the `if/else` statements.
>
> Also be aware that just like any other block, a variable declared as part of an `if` statement will shadow variables with the same name that are declared in the containing blocks.

## `for`, Four Ways

### The Complete `for` Statement

### The Condition-Only `for` Statement

### The Infinite `for` Statement

### `break` and `continue`

### The `for-range` Statement

### Labeling Your `for` Statements

### Choosing the Right `for` Statement

## `switch`

## Blank Switches

## Choosing Between `if` and `switch`

## `goto`-Yes, `goto`

## Wrapping Up
