# Blocks, Shadows, and Control Structures

<!--toc:start-->
- [Blocks, Shadows, and Control Structures](#blocks-shadows-and-control-structures)
  - [Blocks](#blocks)
    - [Shadowing Variables](#shadowing-variables)
    - [The Universe Block](#the-universe-block)
  - [`if`](#if)
  - [`for`, Four Ways](#for-four-ways)
    - [The Complete `for` Statement](#the-complete-for-statement)
    - [The Condition-Only `for` Statement](#the-condition-only-for-statement)
    - [The Infinite `for` Statement](#the-infinite-for-statement)
    - [`break` and `continue`](#break-and-continue)
    - [The `for-range` Statement](#the-for-range-statement)
      - [Iterating over maps](#iterating-over-maps)
    - [Labeling Your `for` Statements](#labeling-your-for-statements)
    - [Choosing the Right `for` Statement](#choosing-the-right-for-statement)
  - [`switch`](#switch)
  - [Blank Switches](#blank-switches)
  - [Choosing Between `if` and `switch`](#choosing-between-if-and-switch)
  - [`goto`-Yes, `goto`](#goto-yes-goto)
  - [Wrapping Up](#wrapping-up)
<!--toc:end-->

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
In Go `for` is the *only* looping keyword in the language and it can be used in four different formats:
- A complete, C-style `for`
- A condition-only `for`
- An infinite `for`
- `for-range`

### The Complete `for` Statement
```go 
for i := 0; i < 10; i++ {
    fmt.Println(i)
}
```

Just like the `if` statement, there are no parenthesis around the parts of the `for` statement. There are three parts, separated by semicolons. The first is an initialization that sets one or more variables before the loop begins. There are two important details to remember about the initialization section. First, you *must* use `:=` to initialize the variables, `var` is *not* legal here. Second, just like variable declaration in `if` statements you can shadow a variable here.

The second part is the comparison. This must be an expression that evaluates to a `bool`. It is checked immediately *before* the loop body runs, after the initialization, and after the loop reaches the end. 

The last part of a standard `for` statement is the increment. This is usually something like `i++` here, but any assignment is valid. It runs immediately after each iteration of the loop, before the condition is evaluated.

### The Condition-Only `for` Statement
Go allows you to leave off both the initialization and the increment in a `for` statement so that it functions like the `while` statement found in many other languages:

```go 
i := 1 
for i < 100 {
    fmt.Println(i)
    i = i * 2
}
```

### The Infinite `for` Statement
Go has a version of a `for` loop that loops forever:

```go 
package main

import "fmt"

func main() {
    for {
        fmt.Println("Hello")
    }
}
```

This program will infinitely print out "Hello", press Ctrl-C to stop the program execution.

### `break` and `continue`
To break out of an infinite `for` loop you can use the `break` statement. It exits the loop immediately and can be used with any `for` statement,not just the infinite `for` statement.

> There is no Go equivalent of the `do` keyword in Java or C. If you want to iterate at least once, the cleanest way is to use an infinite `for `loop that ends with an `if` statement:
> ```go 
> for {
>   // things to do in the loop
>    if !CONDITION {
>        break 
>    }
>}
>```

Go also includes the `continue` keyword, which skips over the rest of the body of a `for` loop and proceeds directly to the next iteration. Technically, you don't need a continue statement. You could write code like this:

```go 
for i := 1, i <= 100; i++ {
    if i%3 == 0 {
        if i%5 == 0 {
            fmt.Println("FizzBuzz")
        } else {
            fmt.Println("Fizz")
        }
    } else if i%5 == 0 {
        fmt.Println("Buzz")
    } else {
        fmt.Println(i)
    }
}
```

This is not idiomatic, Go encourages short `if` statement bodies, as left-aligned as possible. Nested code can be difficult to follow, using a `continue` statement makes it easier to understand what's going on:

```go 
for i := 1; i <= 100; i++ {
    if i%3 == 0 && i%5 == 0 {
        fmt.Println("FizzBuzz")
        continue
    }
    if i%3 == 0 {
        fmt.Println("Fizz")
        continue
    }
    if i%5 == 0 {
        fmt.Println("Buzz")
        continue
    }
    fmt.Println(i)
}
```

### The `for-range` Statement
The `for-range` format is for iterating over elements in some of Go's built-in types and resembles iterators found in other languages. This section will cover how to use a `for-range` loop with strings, arrays, slices, and maps. Once we cover channels in [section 10](./10_concurrency_in_go.md) we will cover how to use them with `for-range` loops as well.

> You can only use `for-range` to iterate over the built-in compound types and user-defined types that are based on them.

```go 
evenVals := []int{2, 4, 6, 8, 10, 12}
for i, v := range evenVals {
    fmt.Println(i, v)
}
```

This code provides the following output:

```shell
0 2
1 4 
2 6 
3 8 
4 10 
5 12
```

The `for-range` loop returns two loop variables. The first variable is the position in the data structure being iterated, the second is the value at that position. The idiomatic names for the two loop variables depends on what is being looped over. For an array, slice, or string, and `i` for *index* is commonly used and when iterating through a map, `k` (for *key*) is used instead. 

The second variable is frequently called `v` for *value*, but it sometimes given a name based on the type of the values being iterated.

If you don't need to access the key, use an underscore(_) as the variable's name. This tells Go to ignore the value.

```go 
evenVals := []int{2, 4, 6, 8, 10, 12}
for _, v := range evenVals {
    fmt.Println(v) // Will only print the value not the index
}
```

If you want the key but don't want the value you can just leave off the second variable:

```go 
uniqueNames := map[string]bool{"Fred": true, "Raul": true, "Wilma": true}
for k := range uniqueNames {
    fmt.Println(k)
}
```

The most common reason for iterating over the key is when a map is being used as a set. If you find yourself using this format for an array or slice, there's a good chance that you have chosen the wrong data structure and should consider refactoring.

#### Iterating over maps
There's something interesting about how a `for-range` loop iterates over a map:

```go 
m := map[string]int{
    "a": 1,
    "c": 3,
    "b": 2,
}

for i := 0; i < 3; i++ {
    fmt.Println("Loop", i)
    for k, v := range m {
        fmt.Println(k, v)
    }
}
```

When you run this program, the output varies. Here is one possibility:

```shell 
Loop 0
c 3
b 2
a 1
Loop 1
a 1 
c 3 
b 2 
Loop 2 
b 2 
a 1 
c 3 
```

The order of the keys and values can vary. This is actually a security feature. In earlier versions of Go the iteration order for keys in a map was usually the same if you inserted the same items into a map. This caused two problems:
- People would write code that assumed that the order was fixed which could break at weird times.
- If maps always hash items to the exact same values, and you know that a server is storing some user data in a map, you can actually slow down a server with an attack called *Hash DoS* by sending it specially crafted data where all of the keys hash to the same bucket.

To prevent both of these problems, the Go team made two changes to the map implementation. First they modified the hash algorithm for maps to include a random number that's generated every time a map variable is created. Next they made the order of `for-range` iterations over a map vary a bit each time they are looped over.

> There is an exception to this rule. To make debugging and logging of maps easier the formatting functions (like `fmt.Println`) always output maps with their keys in ascending sorted order.

### Labeling Your `for` Statements
By default, the `break` and `continue` keywords apply to the `for` loop that directly contains them. If you have nested `for` loops and you want exit or skip over an iteration of an outer loop you would need to use a label:

```go 
func main() {
    samples := []string{"hello", "apple_n!"}
outer:
    for _, sample := range samples {
        for i, r := range sample {
            fmt.Println(i, r, string(r))
            if r == 'l' {
                continue outer
            }
        }
        fmt.Println()
    }
}
```

Notice that the label `outer` is indented by `go fmt` to the same level as the surrounding function. This is done for all labels to make them easier to notice. This program gives the following output:

```go 
0 104 h 
1 101 e 
2 108 l
0 97 a 
1 112 p 
2 112 p 
3 108 l
```

Nested `for` loops with labels are rare. The most common use to implement algorithms similar to the pseudocode below:

```go 
outer:
for _, outerVal := range outerValues {
    for _, innerVal := range outerVal {
        // process innerVal 
        if invalidSituation(innerVal) {
            continue outer
        }
    }
    // here we have code that runs only when all of the 
    // innverVal values were successfully processed
}
```

### Choosing the Right `for` Statement
Most of the time you're going to use the `for-range` format. It is the best way to walk through a string, since it properly gives you back runes instead of bytes. It also works well iterating through slices, maps, and as we'll see in [section 10](./10_concurrency_in_go.md) that channels work naturally with `for-range` as well.

> Favor a `for-range` loop when iterating over all the contents of an instance of one of the built-in compound types. 

The best place for it is when you aren't iterating from the first element to the last element in a compound type. 

The remaining two `for` statement formats are used less frequently. The condition only `for` loop is, like the `while` loop it replaces, useful when you are looping based on a calculated value.

The infinite `for` loop is useful in some situations. There should always be a `break` somewhere within the body of the `for` loop since it's rare that you want to loop forever. Real-world programs should bound iteration and fail gracefully when operations cannot be completed.

## `switch`
At first glance, `switch` statements in Go don't look all that different from how they appear in C/C++ or Java but there are a few surprises:

```go 
words := []string{"a", "cow", "smile", "gopher", "octopus", "anthropologist"}
for _, word := range words {
    switch size := len(word); size {
    case 1, 2, 3, 4:
        fmt.Println(word, "is a short word")
    case 5:
        wordlen := len(word)
        fmt.Println(word, "is exactly the right length:", wordlen)
    case 6, 7, 8, 9:
    default:
        fmt.Println(word, "is a long word!")
    }
}
```

This code has the following output:

```shell
a is a short word!
cow is a short word!
smile is exactly the right length: 5 
anthropologist is a long word!
```

Similar to `if` statements you can declare a variable that's scoped to all of the branches of the `switch` statement. All of the `case` clauses(and the optional `default` clause) are contained inside a set of braces. But you should not put braces around the contents of the `case` clauses. You can have multiple lines inside a `case`(or `default`) clause and they are all considered to be part of the same block.

Inside `case 5:`, we declare `wordLen`, a new variable. Since this a new block, you can declare new variables within it. Just like any other block, any variables declared within a `case` clause's block are only visible within that block.

Other languages often require a `break` statement at the end of every `case` in your `switch` statements but that isn't necessary in Go. By default, cases in `switch` statements in Go don't fall through. 

If cases don't fall through, what do you do if there are multiple values that should trigger the exact same logic? In Go, you separate multiple matches with commas. Additionally, in Go, an empty case means nothing happens.

> For the sake of completeness, Go does include a `fallthrough` keyword which lets one case continue on to the next one. You should think twice before implementing an algorithm that uses it. If you find yourself needing `fallthrough` try restructuring your logic to remove dependencies between cases.

You can switch on any type that can be compared with `==`, which includes all of the built-in types except slices, maps, channels, functions, and structs that contain fields of these types.

Even though you don't need a `break` statement at the end of each `case` clause, you can use them in situations where you want to exit early from a `case`. However, if you need to do this it might indicate that you are doing something too complicated and you should consider a refactor.

## Blank Switches
Just like Go allows you to leave out parts from a `for` statement's declaration, you can write a `switch` statement that doesn't specify the value that you're comparing against. This is called a *blank switch*. A regular `switch` only allows you to compare for equality, a blank switch allows you to use any boolean comparison for each `case`:

```go 
words := []string{"hi", "salutations", "hello"}
for _, word := range words {
    switch wordLen := len(word); {
    case wordLen < 5:
        fmt.Println(word, "is a short word!")
    case wordLen > 10:
        fmt.Println(word, "is a long word!")
    default:
        fmt.Println(word, "is exactly the right length")
    }
}
```

This has the following output:

```shell 
hi is a short word!
salutations is a long word!
hello is exactly the right length
```

Just like a regular `switch` you can optionally include a short variable declaration as part of your blank `switch`. Unlike a regular `switch`, you can write logical tests for your cases. 

## Choosing Between `if` and `switch`
As a matter of functionality there isn't a lot of difference between a series of `if/else` statements and a blank `switch` statement. Both allow a series of comparisons, so how do you decide which one to use? A `switch` statement, even a blank `switch`, indicates that there is some relationship between the values or comparisons in each case.

```go 
switch n := rand.Intn(10); {
case n == 0:
    fmt.Println("That's too low")
case n > 5:
    fmt.Println("That's too big:", n)
default:
    fmt.Println("That's a good number:", n)
}
```

Favor blank `switch` statements over `if/else` chains when you have multiple related cases. Using a `switch` makes the comparisons more visible and reinforces that they are related set of concerns.

## `goto`-Yes, `goto`
The `goto` statement is rarely ever used, many programming languages don't have it and for good reason. It's dangerous because it could jump to nearly anywhere in a program. You could jump into or out of a loop, skip over variable definitions, or into the middle of set of statements in an `if` statement. This makes it difficult to understand what a `goto`-using program does.

In Go, a `goto` statement specifies a labeled line of code and execution jumps to it. However, you can't jump anywhere. Go forbids jumps that skip over variable declarations and jumps that go into an inner or parallel block:  

```go 
func main() {
    a := 10 
    goto skip 
    b := 20 
skip: 
    c := 30 
    fmt.Println(a, b, c) 
    if c > a {
        goto inner
    }
    if a < b {
    inner:
        fmt.Println("a is less than b")
    }
}
```

This program will result in the following errors:

```shell 
goto skip jumps over declaration of b at ./main.go:8:4 
goto inner jumps into block starting at ./main.go:15:11
```

You should rarely ever use `goto`. Labeled `break` and continue statements allow you to jump out of deeply nested loops or skip iteration. This following is a legal `goto` and demonstrates one of the few valid use cases:

```go 
func main() {
    a := rand.Int(10)
    for a < 100 {
        if a%5 == 0 {
            goto done
        }
        a = a*2 + 1
    }
    fmt.Println("do something when the loop completes normally")
done:
    fmt.Println("Do complicated stuff no matter why we left the loop")
    fmt.Println(a)
}
```

In this simple case we have some logic that we don't want to run in the middle of the function, but we do want to run the end of the function. There are ways to do this without `goto`. We could set up a boolean flag or duplicate the complicated code after the `for` loop instead of using `goto`, but this also has drawbacks. Littering your code with boolean flags to control the logic flow is arguably the same functionality as `goto` just more verbose. And duplicating complicated code is problematic because it makes your code harder to maintain. 

## Wrapping Up
This section covered a lot of important topics for writing idiomatic Go. We covered blocks, shadowing, and control structures, and how to use them correctly. In the [next section](./5_functions.md) we'll move on to larger programs, using functions to organize our code.
