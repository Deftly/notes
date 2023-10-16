# Functions

<!--toc:start-->
- [Functions](#functions)
  - [Declaring and Calling Functions](#declaring-and-calling-functions)
    - [Simulating Named and Optional](#simulating-named-and-optional)
    - [Variadic Input Parameters and Slices](#variadic-input-parameters-and-slices)
    - [Multiple Return Values](#multiple-return-values)
    - [Ignoring Returned Values](#ignoring-returned-values)
    - [Named Return Values](#named-return-values)
  - [Functions Are Values](#functions-are-values)
    - [Function Type Declarations](#function-type-declarations)
    - [Anonymous Functions](#anonymous-functions)
  - [Closures](#closures)
    - [Passing Functions as Parameters](#passing-functions-as-parameters)
    - [Returning Functions from Functions](#returning-functions-from-functions)
  - [Keyword `defer`](#keyword-defer)
  - [Go is Call By Value](#go-is-call-by-value)
  - [Wrapping UP](#wrapping-up)
<!--toc:end-->

## Declaring and Calling Functions
We've already seen functions being declared and used. Every program we've written has a `main` function that's the starting point for every Go program, and we've been calling the `fmt.Println` function to display output to the screen. Now let's look at a function that has parameters and returns values:
```go
func div(numerator int, denominator int) int {
  if denominator == 0 {
    return 0
  }
  return numerator / denominator
}
```
A function declaration has four parts: the keyword `func`, the name of the function, the input parameters, and the return type. If a function returns a value, you *must* supply a `return`. If a function returns nothing a `return` statement isn't necessary unless you are exiting a function before the last line.

When you have multiple input parameters of the same type you can write your input parameters like this: `func div(numerator, denominator int) int {`

### Simulating Named and Optional
With one exception that we'll cover later in this section, you must supply all of the parameters for a function. To emulate named and optional parameters, define a struct that has fields that match the desired parameters, and pass the struct to your function.
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
    FirstName: "Joe",
    LastName: "Smith",
  })
}
```
### Variadic Input Parameters and Slices
We've seen that `fmt.Println` allows any number of input parameters, this is done using *variadic parameters*. The variadic parameter *must* be the last(or only) parameter in the input parameter list and is indicated with three dots(...) before the type. The variable that's created within the function is a slice of the specified type:
```go
func addTo(base int, vals ...int) []int {
  out := make([]int, 0, len(vals))
  for _, v := range vals {
    out = append(out, base + v)
  }
  return out
}

func main() {
  fmt.Println(addTo(3)) // []
  fmt.Println(addTo(3, 2)) // [5]
  fmt.Println(addTo(3, 2, 4, 6, 8)) // [5, 7, 9, 11]
  a := []int{4, 3}
  fmt.Println(addTo(3, a...)) // [7, 6]
  fmt.Println(addTo(3, []int{1, 2, 3, 4, 5}...)) // [4, 5, 6, 7, 8]
}
```

### Multiple Return Values
Functions in Go allow for multiple return values, here's a small change to one of our previous examples:
```go
func divAndRemainder(numerator, denominator int) (int, int, error) {
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
Another new concept here is creating and returning an `error`. More on errors in a later section, for now you should know that you use Go's multiple return value support to return an `error` if something goes wrong in your function. If the function completes successfully, we return `nil` for the error's value.

### Ignoring Returned Values
f a function returns multiple values but you don't need to read one or more variables, assign the unused values to the name `_`. For example, if we weren't going to read `remainder` in our previous example we would write the assignment as `result, _, err := divAndRemainder(5, 2)`.

Surprisingly, Go lets you ignore *all* of the return values for a function. You can write `divAndRemainder(5, 2)` without any assignments and the returned values are dropped. We've actually been doing this with `fmt.Println` which returns two values but it's idiomatic to ignore them.

### Named Return Values
Go allows you to specify *names* for return values:
```go
func divAndRemainder(numerator, denominator int) (result, remainder int, err error) {
	if denominator == 0 {
		err = errors.New("cannot divide by zero")
		return result, remainder, err
	}
	result, remainder = numerator/denominator, numerator%denominator
	return result, remainder, err
}
```
When using named return values you are pre-declaring variables that you use within the function to hold the return values. Named return values are initialized to their zero values when created, meaning they can be returned without any explicit use or assignment.

There are some problems with using named return values, the first being that they can be shadowed just like any other variable. The second is that you don't have to return them:
```go
func divAndRemainder(numerator, denominator int) (result, remainder int, err error) {
  result, remainder = 20, 30 // These values aren't returned
  if denominator == 0 {
    return 0, 0, errors.New("cannot divide by zero")
  }
  return numerator / denominator, numerator % denominator, nil // These values are returned
}
```
Named return values give a way to declare an *intent* to use variables to hold the return values, but doesn't *require* you to use them.

Some developers like to use named return parameters because they provide additional documentation but problems with shadowing and ignoring them can make things confusing. We'll cover one situation where they are essential later in this section when we talk about `defer`.

## Functions Are Values
The type of a function is built out of the keyword `func` and the types of the parameters and return values. This combination is called the *signature* of the function. Any function that has the exact same number and types of parameters and return values meets the type signature.

Having functions as values allows us to do some clever things like in the following example:
```go
func add(i, j int) int { return i + j }

func sub(i, j int) int { return i - j }

func mul(i, j int) int { return i * j }

func div(i, j int) int { return i / j }

var opMap = map[string]func(int, int) int{
	"+": add,
	"-": sub,
	"*": mul,
	"/": div,
}

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
> **_NOTE:_** The core logic for this example is relatively short. Of the 22 lines in the `for` loop, 6 of them implement the actual algorithm and the other 16 are error checking and data validation. You might be tempted to skip out on these checks but doing so produces unstable, unmaintainable code. Error handling is what separates the professionals from the amateurs.

### Function Type Declarations
Just like you can use the `type` keyword to define a `struct`, you can use it to define a function type too:
```go
type opFuncType func(int, int) int

var opMap = map[string]opFuncType {
  // same as before
}
```
Declaring a function type can serve as documentation, and another use case which we'll cover when we talk about interfaces.

### Anonymous Functions
You can define new functions within a function and assign them to variables or call them inline. These are known as *anonymous functions*:
```go
func main() {
  for i := 0; i < 5; i++ {
    func(j int) {
      fmt.Println("printing", j, "from inside of an anonymous function")
    }(i)
  }
}
```
This is not something you would normally do. If you are declaring and executing an anonymous function immediately you might as well get rid of the anonymous function and just call the code. Declaring anonymous functions without assigning them to variables is useful when used with `defer` statements and launching goroutines. We'll talk about `defer` later in this section and goroutines in a later section.

## Closures
Functions declared inside of functions are *closures*. This is a computer science term that means functions declared inside of functions are able to access and modify variables declared in the outer function.

One thing that closures allow you to do is limit a function's scope. If a function is only going to be called from one other function, but is called multiple times, you can use an inner function to "hide" the called function. This reduces the number of declarations at the package level.

When closures become really interesting is when they are passed to other functions or returned from a function. This allows you to take the variables within your function and use those variables *outside* of your function.

### Passing Functions as Parameters


### Returning Functions from Functions


## Keyword `defer`

## Go is Call By Value

## Wrapping UP
