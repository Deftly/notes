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
Because functions are values and you can specify the type of a function you can pass functions as parameters into functions. Think about the implications of creating a closure that references local variables and then passing that closure to another function. This pattern can be very useful and appears several time in the standard library.

Let's see how closures are used to sort the same data in different ways:
```go
type Person struct {
  FirstName string
  LastName string
  Age int
}

people := []Person{
  {"Pat", "Patterson", 37},
  {"Tracy", "Bobbert", 23},
  {"Fred", "Fredson", 18},
}
fmt.Println(people)
//sort by last name
sort.Slice(people, func(i int, j int) bool {
  return people[i].LastName < people[j].LastName
})
fmt.Println(people)
//sort by age
sort.Slice(people, func(i int, j int) bool {
  return people[i].Age < people[j].Age
})
fmt.Println(people)
```
> **_NOTE:_** The people slice is changed by the call to `sort.Slice`, we'll cover this briefly in the section [Go is Call By Value](#go-is-call-by-value)

Passing functions as parameters to other functions is often useful for performing different operations on the same kind of data.

### Returning Functions from Functions
You can also return a closure from a function, let's take a look at an example:
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
This gives us the following output:
```shell
0 0
2 3
4 6
```
## Keyword `defer`
Programs often create temporary resources, like files or network connections. These have to be cleaned up regardless of where the function exits or whether it completed successfully or not. In Go, the cleanup code is attached to the function with the `defer` keyword.

As an example we'll write a simple version of `cat`, a Unix utility for printing the contents of a file:
```go
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
There are a lot of new features in this example but for now we'll focus on the parts that relate to using `defer`. In the example we open a file handle. Once we know we have a valid file we will need to close it no matter how we exit the function. To ensure the cleanup code runs, we use the `defer` keyword followed by a function/method call. In this case, we use the `Close` method on the file variable. Normally the function call would run immediately but `defer` delays the invocation until the surrounding function exits.

You can `defer` multiple closures in a Go function. They will run in last-in-first-out order, which is to say the last `defer` registered runs first.

The code within `defer` closures run *after* the return statement. You can supply a function with input parameters to a `defer`, just be sure to remember that any variables passed into a deferred closure aren't evaluated until the closure runs.

Deferred functions can also examine or modify the return values of the surround function by making use of named return values. This allows us to take actions based on an error, in a later section we'll cover a pattern that uses a `defer` to add contextual information to an error returned from a function. Now let's look at an example:
```go
func DoSomeInserts(ctx context.Context, db *sql.DB, value1, value2 string) (err error) {
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
  // use tx to do more database inserts 
  return nil
}
```
In this example we create a transaction to do a series of database inserts. If any of them fail we want to roll back(not modify the database). If all of them succeed we want to commit(store the database changes). We use a closure with `defer` to check if `err` has been assigned a value. If it hasn't, we run a `txCommit()`, not that this can also return an error. If it does, the value `err` is modified and we can call `tx.Rollback()`.

## Go is Call By Value

## Wrapping UP
