# Composite Types
In this section we'll learn about the composite types in Go, the built-in functions that support them, and the best practices for working with them.

## Arrays-Too Rigid to Use Directly
Like most languages Go has arrays. However, arrays are rarely used directly in Go and we'll soon see why. First we'll cover array declaration syntax and use.

All of the elements in the array must of the type that's specified. There are a few different declaration styles. In the first, you specify the size of the array and the type of the elements in the array:

```go
var x [3]int
```

This creates an array of three `int`s. Because no values were specified, all of the positions are initialized to the zero value for an `int`. If you have initial values you can specify them with an *array literal*:

```go 
var x = [3]int{10, 20, 30}
```

If you have a *sparse array*(an array where most elements are set to the zero value), you can specify only the indices with values in the array literal:

```go
var x = [12]int{1, 5: 4, 6, 10: 100, 15} // [1, 0, 0, 0, 0, 4, 6, 0, 0, 0, 100, 15]
```

When using an array literal for initialization you can leave off the number and use `...` instead:

```go
var x = [...]int{10, 20, 30}
```

You can use `==` and `!=` to compare arrays:

```go
var x = [...]int{1, 2, 3}
var y = [3]int{1, 2, 3}
fmt.Println(x == y)
```

Go only has one-dimensional arrays, but you can simulate multidimensional arrays:

```go
var x [2][3]int
```

This declares `x` to be an array of length 2 whose type is an `int` array of length 3. Go doesn't have true matrix support.

Like most languages, arrays in Go are read and written using bracket syntax:

```go
x[0] = 10
fmt.Println(x[2])
```

You cannot read or write past the end of an array or use a negative index. Doing so with a constant or literal index is a compile-time error. An out-of-bounds read or write with a variable index compiles but fails at runtime with a panic. 

The built-in function `len` takes in an array and returns its length:

```go
fmt.Println(len(x))
```


