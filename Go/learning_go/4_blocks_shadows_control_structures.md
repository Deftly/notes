# Blocks, Shadows, and Control Structures
In this section we'll first look at blocks and how they control when an identifier is available. Then we'll look at Go's control structures: `if`, `for`, and `switch`. 

## Blocks
Go lets you declare variables outside of function, as parameters to functions,a nd as local variables within functions. Each place where a declaration occurs is called a *block*. Variables, constants, types, and functions declared outside of any functions are placed in the *package* block. The `import` statements(more detail about them in [section 9](./9_modules_packages_imports.md)) define names for other packages that are valid for the file that contains the `import` statement. These names are in the *file* block. All variables defined at the top level of a function, including the parameters to a function, are in a block. Within a function, every set of braces({}) defines another block.

You can access any identifier in any outer block from within any inner block. So what happens when you have a declaration with the same name as an identifier in a containing block? When this happens you *shadow* the identifier created in the outer block.

### Shadowing Variables
Let's take a look at an example of shadowing variables:

```Go
func main() {
	x := 10
	if x > 5 {
		fmt.Println(x) // 10
		x := 5
		fmt.Println(x) // 5
	}
	fmt.Println(x)   // 10
}
```

A shadowing variables is a variable with the same name as a variable in a containing block. As long as the shadowing variable exits, you cannot access the shadowed variable.

```Go
func main() {
	x := 10
	if x > 5 {
		x, y := 5, 20
		fmt.Println(x, y) // 5 20
	}
	fmt.Println(x)      // 10
}
```

In the above example, although there was an existing definition of `x` in an outer block, `x` was still shadowed within the `if` statement. This is because `:=` only reuses variables that are declared in the current block. When using `:=`, make sure that you don't have any variables from an outer scope on the left hand side unless you intend to shadow them.

You also need to be careful to ensure that you don't shadow a package import like in the example below:

```Go
func main() {
  x := 10
  fmt.Println(x)
  fmt := "oops"
  fmt.Println(fmt) // fmt.Println undefined (type string has no field or method Println)
}
```

Once the local variable `fmt` is declared, it shadows the package named `fmt` in the file block, making it impossible to use the `fmt` package for the rest of the `main` function.

### The Universe Block
There's actually one more block that is a little weird: the universe block. Go is a small language with only 25 keywords, but built-in types(like `int` and `string`), constants(like `true` and `false`), and functions(like `make` or `close`) aren't included in that list. Neither is `nil`, so what are they?

Go considers these *predeclared identifiers* and defines them in the universe block, which contains all other blocks. This means that they can be shadowed in other scopes like in the example below:

```Go
fmt.Println(true) // true
true := 10
fmt.Println(true) // 10
```

You must be very careful to never redefine any of the identifiers in the universe block. If you accidentally do so you will get some very strange behavior. If you're lucky you'll get compilation failures, if not you may have a hard time tracking down the source of the bugs. 

## if
The `if` in Go is much like the `if` statement in most programming languages:

```Go
n := rand.Intn(10)
if n == 0 {
  fmt.Println("That's too low")
} else if n > 5 {
  fmt.Println("That's too big:", n)
} else {
  fmt.Println("That's a good number:", n)
}
```

The most visible difference between `if` statements in Go and other languages is that you don't put parenthesis around the condition. But there's another feature that Go adds to `if` to better manage your variables.

We mentioned earlier that any variable declared within the braces of an `if` or `else` only exists within that block. What Go adds is the ability to declare variables that are scoped to the condition and to both the `if` and `else` blocks:

```Go
if n := rand.Intn(10); n == 0 {
  fmt.Println("That's too low")
} else if n > 5 {
  fmt.Println("That's too big:", n)
} else {
  fmt.Println("That's a good number:", n)
}
```

This special scope is very handy, it lets you create variables that are available only where they are needed. Once the series of `if/else` statements ends, `n` is undefined. 

> **_NOTE:_** Technically, you can put any *simple statement* before the comparison in an `if` statement. This includes a function call that doesn't return a value or assigning a new value to an existing variable. But you shouldn't do this, only use it to define new variables that are scoped to the `if/else` statements; anything else reduces your code clarity.

## for, Four Ways
In Go `for` is the *only* looping keyword in the language, but it can be used in four different formats:
- A complete, C-style `for`
- A condition-only `for`
- An infinite `for`
- `for-range`

### The Complete for Statement
This will be familiar if you are coming from C, Java, or JavaScript:

```Go
for i := 0; i < 10; i++ {
  fmt.Println(i)
}
```

Just like the `if` statement there are no parenthesis around the parts of the `for` statement, otherwise it should look very familiar. There are three parts separated by semicolons. The first in an initialization that sets one or more variables before the loop begins. You *must* use `:=` to initialize the variables, `var` is not legal here, and just like variable declarations in an `if` statement, you can shadow variables here.

The second part is the comparison. This expression must evaluate to a `bool`. It is checked immediately before the loop body runs, after the initialization, and after the loop reaches the end.

The last part of any standard `for` is the increment. Usually it's something like i++ but any assignment is valid and it runs immediately after each iteration of the loop, before the condition is evaluated.

### The Condition-Only for Statement
Go allows you to leave off both the initialization and increment in a `for` statement. This function like the `while` statement found in many other languages:

```Go
i := 1
for i < 100 {
  fmt.Println(i)
  i = i * 2
}
```

### The Infinite for Statement
The third `for` statement does away the condition too:

```Go
for {
  fmt.Println("Hello")
}
```

To get out of an infinite loop without using the keyboard you need a `break` statement.

There is no Go equivalent of the `do` keyword in other languages. So to iterate at least once, the cleanest way is to use an infinite `for` loop that ends with an `if` statement:

```Go
for {
  //Things to do in the loop
  if !CONDTION {
    break
  }
}
```

Go also includes the `continue` keyword which skips over the rest of the body of a `for` loop and proceeds directly to the next iteration:

```Go
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

### The for-range Statement
The fourth `for` statement is for iterating over elements in some of Go's built-in types, and resembles iterators found in other languages. We'll see how we can use it with strings, arrays, slices, and maps. In [section 10](./10_concurrency_in_go.md) we'll see how they can be used with channels as well.

> **_NOTE:_** You can only use a `for-range` loop to iterate over the built-in compound types and user-defined types that are based on them.

```Go
evenVals := []int{2, 4, 6, 8, 10, 12}
for i, v := range evenVals {
  fmt.Println(i, v)
}
```

```Shell
0 2
1 4
2 6
3 8
4 10
5 12
```

What makes a `for-range` loop interesting is that you get two loop variables. The first variable is the position in the data structure being iterated, while the second is the value at that position. Below are some more examples:

```Go
evenVals := []int{2, 4, 6, 8, 10, 12}
for _, v := range evenVals { // Ignoring the index
  fmt.Println(v)
}

uniqueNames := map[string]bool{"Fred": true, "Raul": true, "Wilma": true}
for k := range uniqueNames { // Ignoring the key
  fmt.Println(k)
}
```

#### Iterating over maps
When a `for-range` loop iterates over a map the order of the keys and values can vary. This is actually a security feature. In earlier Go version, the iteration order for keys in a map was usually(but not always) the same if you inserted the same items into a map. This lead to 2 problems:
- People would write code that assumed the order was fixed, this would break at weird times.
- If maps always hash items to the exact same values, you can actually slow down a server with an attack called *Hash DoS* by sending specially crafted data where all the keys has to the same bucket.

To prevent these problems, the Go team modified the hash algorithm for maps to include a random number that's generated every time a map variable is created. They also made the order of a `for-range` iteration over a map vary a bit each time the map is looped over making it much harder to implement a Hash DoS attack.

#### Iterating over strings
```Go
samples := []string{"hello", "apple_π!"}
for _, sample := range samples {
  for i, r := range sample {
    fmt.Println(i, r, string(r))
  }
}
fmt.Println()
```

The output for the word "hello" has no surprises. In the first column we have the index, in the second the numeric value of the letter, and in the third we have the numeric value of the letter type converted to a string.

```Shell
0 104 h
1 101 e
2 108 l
3 108 l
4 111 o
```

The result for "apple_π" is more interesting:

```Shell
0 97 a
1 112 p
2 112 p
3 108 l
4 101 e
5 95 _
6 960 π
8 33 !
```

Notice that the first column skips the number 7 and that the value at position 6 is 960. That's much larger than what can fit in a byte.

What we see is the special behavior from iterating over a string with the `for-range` loop. It iterates over the *runes* not the *bytes*. Whenever the `for-range` loop encounters a multi-byte rune in a string it converts the UTF-8 representation into a single 32-bit number and assigns it to the value. The offset is incremented by the number of bytes in the rune.

#### The for-range value is a copy
You should be aware that each time the `for-range` loop iterates over your compound type, it *copies* the value from the compound type to the value variable. *Modifying the value variable will not modify the value in the compound type*.

### Labeling Your for Statements
By default, the `break` and `continue` keywords apply to the `for` loop that directly contains them. We can change this behavior using labels:

```Go
func main() {
  samples := []string{"hello", "apple_π!"}
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

Labels like `outer` are always indented to the same level as the braces for the block. This makes them easier to notice. Below is the output:

```Shell
0 104 h
1 101 e
2 108 l
0 97 a
1 112 p
2 112 p
3 108 l
```

Nest `for` loops with labels are rare. The most common usage is to implement algorithms similar to the follow pseudocode:

```Go
outer:
  for _, outerVal := range outerValues {
    for _, innerVal := range outerVal {
      // process innerVal
      if invalidSituation(innerVal) {
        continue outer
      }
    }
    // here we have code that runs only when all of the
    // innerVal values were successfully processed
  }
```

## Choosing the Right for Statement
Most of the time you'll use the `for-range` format. It is the best way to walk through a string, since it properly gives you back runes instead of bytes. It also works with slices, maps and channels. 

The complete `for` loop is best when you aren't iterating from the first element to the last element in a compound type. 

The remaining 2 `for` statement formats are used less frequently. The condition only `for` loop is, like the while loop it replaces, useful when you are looping based on a calculated value.

## switch





