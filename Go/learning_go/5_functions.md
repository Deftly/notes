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
    - [Returning Functions from Functions](#returning-functions-from-functions)
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
We've been using `fmt.Println` to print results to the screen and you've probably noticed that it allows any number of input parameters, how does it do that? Go supports *variadic parameters*. The variadic parameter *must* be the last(or only) parameter in the input parameter list. This is indicated with `...` before the type. The variable that's created within the function is a slice of the specified type and can be used like any other slice. 

```go 
func addTo(base int, vals ...int) []int {
    out := make([]int, 0, len(vals))
    for _, v := range vals {
        out = append(out, base+v)
    }
}

func main() {
    fmt.Println(addTo(3))                           // []
    fmt.Println(addTo(3, 2))                        // [5]
    fmt.Println(addTo(3, 2, 4, 6, 8))               // [5 7 9 11]
    a := []int{4, 3}
    fmt.Println(addTo(3, a...))                     // [7 6]
    fmt.Println(addTo(3, []int{1, 2, 3, 4, 5}...))  // [4 5 6 7 8]
}
```

You can supply however many values you want for variadic parameter, or no values at all. Since the variadic parameter is converted to a slice, you can supply a slice as the input. When doing this you must put `...` after the variable or slice literal, not doing so will result in a compile-time error.

### Multiple Return Values 
A major difference between Go and other languages is that Go allows for multiple return values. Let's make a small update to our previous division program:

```go 
func divAndRemainder(numerator int, denominator int) (int, int, error) {
    if denominator == 0 {
        return 0, 0, errors.New("cannot divide by zero")
    }
    return numerator / denominator, numerator % denominator, nil
}

func main() {
    result, remainder, err := divAndRemainder(5, 2)
    if err != nil {
        fmt.Println(err)
        os.Exit(1)
    }
    fmt.Println(result, remainder)
}
```

When a Go function returns multiple values, the types of the return values are listed in parentheses, separated by commas. 

Something else we haven't seen yet is creating and returning an `error`. To learn more about errors read [section 8](./8_errors.md). Go's multiple return value support is used to return an `error` if something goes wrong in a function. If the function completes successfully we return `nil` for the error's value. By convention, the `error` is always the last(or only) value returned from a function.

### Multiple Return Values Are Multiple Values 
In Python, functions can return a tuple that's optionally destructured if the tuple's values are assigned to multiple variables:

```python 
>>> def div_and_remainder(n, d):
        if d == 0:
            raise Exception("cannot divide by zero")
        return n / d, n % d
>>> v = div_and_remainder(5, 2)
>>> v 
(2.5, 1)
>>> result, remainder = div_and_remainder(5, 2)
>>> result 
2.5 
>>> remainder 
1
```

That isn't how Go works, You must assign each value returned from a function. If you try to assign multiple return values to one variable you will get a compile-time error.

### Ignoring Returned Values 
If a function returns multiple values, but you don't need to read one or more of the values, assign the unused values to the name `_`. For example, if we weren't going to read `remainder` from our previous example, we would write the assignment as: `result, _, err := divAndRemainder(5, 2)`

Go also lets you implicitly ignore *all* of the return values for a function. We have actually been doing this since our earliest examples with `fmt.Println` which returns two values(the number of bytes written and any write errors encountered), but it is idiomatic to ignore them. In almost all other cases, you should make it explicit that you are ignoring return values using underscores.

### Named Returned Values 
Go allows you to specify *names* for your return values, lets rewrite our `divAndRemainder` function using named return values:

```go 
func divAndRemainder(numerator int, denominator int) (result int, remainder int, err error) {
    if denominator == 0 {
        err = errors.New("Cannot divide by zero")
        return result, remainder, err
    }
    result, remainder = numerator/denominator, numerator%denominator
    return result, remainder, err
}

func main() {
    x, y , z := divAndRemainder(5, 2)
    fmt.Println(x, y, z) // 2, 1, nil
}
```

When you supply names to your return values, you are pre-declaring variables that you use within the function to hold the return values. Named return values are initialized to their zero values when created. This means that we can return them before any explicit use or assignment.

While named return values can sometimes help clarify your code, they have a few problems and corner cases. The first problem is shadowing. Just like any other variable you can shadow a named return value. The other problem is that you don't have to return them. Let's look at another variation of `divAndRemainder`:

```go 
func divAndRemainder(numerator, denominator int) (result int, remainder int, err error) {
    result, remainder = 20, 30
    if denominator == 0 {
        return 0, 0, errors.New("Cannot divide by zero")
    }
    return numerator / denominator, numerator % denominator, nil
}
```

Here we assigned values to `result` and `remainder` and then returned different values directly. When we pass 5 and 2 to this function we get this result: `2 1`.

The values from the `return` statement were returned even though they were never assigned to the named return parameters. This is because the Go compiler inserts code that assigns whatever is returned to the return parameters. 

Some developers like to use named return parameters as a form of additional documentation but in general they provide limited value. Shadowing and simply ignoring them can make things confusing. There is one situation where named return parameters are essential, we'll cover this when going over `defer` later in this section.

### Blank Returns-Never Use These!
If you have named return values, you can just write `return` without specifying the values that are returned. This returns the last values assigned to the named return values. Let's rewrite our `divAndRemainder` function one last time, this time using blank returns:

```go 
func divAndRemainder(numerator, denominator int) (result int, remainder int, err error) {
    if denominator == 0 {
        err = errors.New("Cannot divide by zero")
        return
    }
    result, remainder = numerator/denominator, numerator%denominator
    return
}
```

When we have invalid input we return immediately. Since no values were assigned to `result` and `remainder`, their zero values are returned. We also still have to put a `return` at the end of the function.

Most experienced Go developers consider blank returns a bad idea because they make it harder to understand data flow. When you use a blank return, the reader of you code needs to scan back through the program to find the last value assigned to the return parameters to see what is actually being returned.

## Functions are Values
Like in many other languages, functions in Go are values. The type of a function is built out of the keyword `func` and the types of the parameters and return values. This combination is called the *signature* of the function. Any function that has the exact same number and types of parameters and return values meets the type signature.

Having functions as values allows us to do some clever things, such as build a simple calculator using functions as values in a map. First we'll create a set of functions that all have the same signature:

```go 
func add(i, j int) int { return i + j }

func sub(i, j int) int { return i - j }

func mul(i, j int) int { return i * j }

func div(i, j int) int { return i / j }
```

Next we'll create a map to associate a math operator with each function:

```go 
var opMap = map[string]func(int, int) int{
	"+": add,
	"-": sub,
	"*": mul,
	"/": div,
}
```

We can test out the calculator with a few expressions:

```go 
func main() {
	expressions := [][]string{
		{"2", "+", "3"},
		{"2", "-", "3"},
		{"2", "*", "3"},
		{"2", "/", "3"},
		{"2", "%", "3"},
		{"two", "+", "three"},
		{"5"},
	}

	for _, expression := range expressions {
		if len(expression) != 3 {
			fmt.Println("invalid expression:", expression)
			continue
		}
		p1, err := strconv.Atoi(expression[0])
		if err != nil {
			fmt.Println(err)
			continue
		}
		op := expression[1]
		opFunc, ok := opMap[op]
		if !ok {
			fmt.Println("unsupported operator:", op)
			continue
		}
		p2, err := strconv.Atoi(expression[2])
		if err != nil {
			fmt.Println(err)
			continue
		}
		result := opFunc(p1, p2)
		fmt.Println(result)
	}
}
```

When we run the program we get the following output:

```
5
-1
6
0
unsupported operator: %
strconv.Atoi: parsing "two": invalid syntax
invalid expression: [5]
```

> The core logic for the above example is quite short. Of the 22 lines inside the `for` loop, 6 of them implement the actual algorithm and the other 16 are error checking and data validation. You might be tempted to skip out on validating data or checking errors, but doing so produces unstable, unmaintainable code. Error handling is what separates the professionals from the amateurs. 

### Function Type Declarations
Just like you can use the `type` keyword to define a `struct`, you can use it to define a function type(we'll go into more detail on type declarations in [section 7](./7_types_methods_interfaces.md)):

```go 
type opFuncType func(int,int) int 
```

We can then rewrite the `opMap` declaration from earlier to look like this:

```go 
var opMap = map[string]opFuncType {
    // same as before
}
```

What's the advantage of declaring a function type? One use is documentation. It's useful to give something a name if you are going to refer to it multiple times. We'll see another use in [section 7](./7_types_methods_interfaces.md) when we talk about interfaces.

### Anonymous Functions
In addition to assigning functions to variables you can also define new functions within a function and assign them to variables.

These inner functions are know as *anonymous functions* because they don't have a name. You don't even have to assign them to a variable, you can write them inline and call them immediately:

```go 
func main() {
    for i := 0; i < 5; i++ {
        func(j int) {
            fmt.Println("printing", j, "from inside of an anonymous function")
        }(i)
    }
}
```

You declare an anonymous function with the keyword `func` followed immediately by the input parameters, the return values, and the opening brace. It is a compile-time error to try and put a function name between `func` and the input parameters.

The above program gives us the following output:

```shell 
printing 0 from inside of an anonymous function
printing 1 from inside of an anonymous function
printing 2 from inside of an anonymous function
printing 3 inside of an anonymous function
printing 4 from inside of an anonymous function
```

This isn't something you would normally do. If you are declaring and executing an anonymous function immediately, you might as well get rid of the anonymous function and just call the code. There are two situations where declaring anonymous functions without assigning them to variables is useful: `defer` statements and launching goroutines. We'll talk about `defer` statements later in this section and goroutines in [section 10](./10_concurrency_in_go.md)

## Closures
Functions declared inside of functions are referred to as *closures*. This is a computer science term that means that functions declared inside of functions are able to access and modify variables declared in the outer function.

What benefit do we get from making mini-functions within larger functions and why does Go have this feature?

One thing that closures allow you to do is limit a function's scope. If a function is only going to be called from one other function, but it's called multiple times, you can use an inner function to "hide" the called function. This reduces the number of declarations at the package level and can make it easier to find an unused name.

Closures really become interesting when they are passed to other functions or are returned from a function. They allow you to take variables within your function and use those values *outside* of your function.

### Passing Functions as Parameters
Since functions are values you can specify the type of a function using its parameter and return types, and then pass functions as parameters into functions.

Let's look at an example with sorting slices. The `sort` package in the standard library has a function called `sort.Slice`. This function takes any slice and a function that is used to sort the slice that's passed in:

```go 
type Person struct {
	FirstName string
	LastName  string
	Age       int
}

people := []Person{
    {"Pat", "Patterson", 37},
    {"Tracy", "Bobbert", 23},
    {"Fred", "Fredson", 18},
}
fmt.Println(people) // [{Pat Patterson 37} {Tracy Bobbert 23} {Fred Fredson 18}]

// sort by last name
sort.Slice(people, func(i int, j int) bool {
    return people[i].LastName < people[j].LastName
})
fmt.Println(people) // [{Tracy Bobbert 23} {Fred Fredson 18} {Pat Patterson 37}]

// sort by age
sort.Slice(people, func(i int, j int) bool {
    return people[i].Age < people[j].Age
})
fmt.Println(people) // [{Fred Fredson 18} {Tracy Bobbert 23} {Pat Patterson 37}]
```

The closure that's passed to `sort.Slice` has two parameters, `i` and `j`, but within the closure, we can refer to `people` so we can sort it by the `LastName` field. In computer science terms, `people` is *captured* by the closure. We do a similar sorting afterwards using the `Age` field.

Not that the `people` slice is changed by the call to `sort.Slice`. We'll cover this briefly in [Go is Call By Value](#go-is-call-by-value) and in more detail in the [next section](./6_pointers.md).

### Returning Functions from Functions 
Not only can we use a closure to pass some function state to another function, you can also return a closure from a function:

```go 
func makeMult(base int) func(int) int {
    return func(factor int) int {
        return base * factor
    }
}

func main() {
    twoBase := makeMult(2)
    threeBase := makeMult(3)
    for i := 0; i < 3; i++ {
        fmt.Println(twoBase(i), threeBase(i))
    }
}
```

Running this program produces the following output:

```shell 
0 0
2 3 
4 6 
```

Closures are quite useful, we've already seen how they are used to sort slices. A closure is also used to efficiently search a sorted slice with `sort.Search`. As for returning closures, later on we'll see this pattern used when we build middleware for a web server. Go also uses closures to implement resource cleanup, via the `defer` keyword.

## `defer`
Programs often create temporary resources, like files or network connections, that need to be cleaned up. This must occur no matter how many exit points a function has, or whether a function completed successfully or not. In Go, the cleanup code is attached to the function with the `defer` keyword.

We'll look at how to use `defer` to release resources by writing a simple version of `cat`, the Unix utility for printing the contents of a file:

```go 
import (
	"io"
	"log"
	"os"
)

func main() {
	if len(os.Args) < 2 {
		log.Fatal("no file specified")
	}
	f, err := os.Open(os.Args[1])
	if err != nil {
		log.Fatal(err)
	}
	defer f.Close()

	data := make([]byte, 2048)
	for {
		count, err := f.Read(data)
		os.Stdout.Write(data[:count])
		if err != nil {
			if err != io.EOF {
				log.Fatal(err)
			}
			break
		}
	}
}
```

This example introduces some new features that we will cover in detail in later sections.

First, we make sure that a file name was specified on the command line by checking the length of `os.Args`, a slice in the `os` package that contains the name of the program launched and the arguments passed to it. If the argument is missing we use the `Fatal` function in the `log` package to print a message and exit the program. Next we get a read-only file handle with the `Open` function in the `os` package. If there is an error opening the file we print the error message and exit the program. We'll talk more about errors in [section 8](./8_errors.md).

Once we know we have a file handle, we need to close it after we use it no matter how we exit the function. To ensure this happens we use the `defer` keyword, followed by a function or method call. In this case, we use the `Close` method on the file variable(We'll look at methods in [section 7](./7_types_methods_interfaces.md)). Normally a function will run immediately, but `defer` delays the invocation until the surrounding function exits.

We read from a file handle by passing a slice of bytes into the `Read` method on a file variable. `Read` returns the number of bytes that were read into the slice and an error. If there is an error we'll check to see if it's an end-of-file marker. If it is we'll `break` to exit the `for` loop. For all other errors, we report it and exit immediately using `log.Fatal`.

There are a few more things to know about `defer`. First, you can `defer` multiple closures in a Go function. They will run in last-in-first-out order; the last `defer` registered runs first.

The code within `defer` closures run *after* the return statement. We can supply a function with input parameters to `defer`, and just as `defer` doesn't run immediately, any variables passed into a deferred closure aren't evaluated until the closure runs.

 We might wonder if there's a way for a deferred function to examine or modify the return values of its surrounding function. There is, and it's the best reason to use named return values. This can let us take action based on an error. In [section 8](./8_errors.md) we'll cover a patter that uses `defer` to add contextual information to an error returned from a function. For now, let's look at a way to handle database transaction cleanup using named return values and `defer`: 

```go 
func DoSomeInserts(ctx context.Context, db *sql.DB, value1, value2 string) (err, error) {
    tx, err := db.BeginTx(ctx, nil)
    if err != nil {
        return err
    }
    defer func() {
        if err == nil {
            err = tx.Commit()
        }
        if err != nil {
            tx.Rollback()
        }
    }()
    _, err = tx.ExecContext(ctx, "INSERT INTO FOO (val) values $1", value1)
    if err != nil {
        return err
    }
    // use tx to do more database inserts here
    return nil
}
```

In this example function, we create a transaction to do a series of database inserts. If any of them fail, we want to roll back. If all of them succeed, we want to commit. We can use a closure with `defer` to check if `err` has been assigned a value. If it hasn't, we run a `tx.Commit()`, which could also return an error. If it does, the value of `err` is modified. If any database interaction returned an error, we call `tx.Rollback()`.

## Go Is Call By Value


## Wrapping Up
