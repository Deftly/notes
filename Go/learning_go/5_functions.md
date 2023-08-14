# Functions

<!--toc:start-->
- [Functions](#functions)
  - [Declaring and Calling Functions](#declaring-and-calling-functions)
    - [Simulating Named and Optional Parameters](#simulating-named-and-optional-parameters)
    - [Variadic Input Parameters and Slices](#variadic-input-parameters-and-slices)
    - [Multiple Return Values](#multiple-return-values)
    - [Multiple Return Values Are Multiple Values](#multiple-return-values-are-multiple-values)
    - [Ignoring Returned Values](#ignoring-returned-values)
    - [Named Returned Values](#named-returned-values)
    - [Blank Returns-Never Use These!](#blank-returns-never-use-these)
  - [Functions are Values](#functions-are-values)
    - [Function Type Declarations](#function-type-declarations)
    - [Anonymous Functions](#anonymous-functions)
  - [Closures](#closures)
    - [Passing Functions as Parameters](#passing-functions-as-parameters)
    - [Returning Functions from Functions]
  - [`defer`](#defer)
  - [Go Is Call By Value](#go-is-call-by-value)
  - [Wrapping Up](#wrapping-up)
<!--toc:end-->

In this section we're going to learn how to write functions in Go and see all of the interesting things we can do with them.

## Declaring and Calling Functions
We've already seen functions being declared and used. Every program we've written has a `main` function that's the starting point for every Go program, and we've been calling the `fmt.Println` function to display output. Let's look at a function that has parameters and returns values:

```go 
func div(numerator int, denominator int) int {
    if denominator == 0 {
        return 0
    }
    return numerator / denominator
}
```

A function declaration has four parts: the keyword `func`, the name of the function, the input parameters, and the return type. Go is a typed language so you must specify the types of the parameters.

Just like other languages, Go has a `return` keyword for returning values from a function. If a function returns a value, you *must* supply a `return`. If a function returns nothing, a `return` isn't needed unless you want to exit from the function before the last line.

> When you have multiple input parameters of the same type, you can write your input parameters like this:
>
> `func div(numerator, denominator int) int {`

### Simulating Named and Optional Parameters
Go doesn't have named and optional input parameters. With one exception that we will cover later, you must supply all of the parameters for a function. If you want to emulate named and optional parameters, define a struct that has fields that match the desired parameters, and pass the struct to your function:

```go 
type MyFuncOpts struct {
    FirstName string 
    LastName string 
    Age int
}

func MyFunc(opts MyFuncOpts) error {
    // do something
}

func main() {
    MyFunc(MyFuncOpts {
        LastName: "Patel",
        Age: 50,
    })
    MyFunc(MyFuncOpts {
        FirstName: "Jose",
        LastName: "Smith",
    })
}
```

In practice not having named and optional parameters isn't a limitation. A function shouldn't have more than a few parameters, and named and optional parameters are mostly useful when a function has many inputs. If you find yourself in that situation there's a good chance your function is too complicated.

### Variadic Input Parameters and Slices
We've been using `fmt.Println` to print results to the screen and you've probably noticed that it allows any number of input parameters, how does it do that? Go supports *variadic parameters*

### Multiple Return Values 

### Multiple Return Values Are Multiple Values 

### Ignoring Returned Values 

### Named Returned Values 

### Blank Returns-Never Use These!

## Functions are Values

### Function Type Declarations

### Anonymous Functions

## Closures

### Passing Functions as Parameters

### Returning Functions from Functions 

## `defer`

## Go Is Call By Value

## Wrapping Up
