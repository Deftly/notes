# Composite Types
In the previous section we looked at simple types: numbers, booleans, and strings. In this section we'll learn about composite types in Go, the built-in functions that support them, and how to best use them.

## Arrays - Too Rigid to Use Directly
All elements in an array must be of the type that's specified, and there are a few different declaration styles. 

```Go
var x [3]int // Since no values specified all positions are initialized with 0
var y = [3]int{10, 20, 30} // initial values set with array literal
var z = [12]int{1, 5: 4, 6, 10: 100, 15} // sparse array [1 0 0 0 0 4 6 0 0 0 100 15]
var a = [...]int{10, 20, 30} // Can use ... instead of a number when using an array literal
```

You can use `==` and `!=` to compare arrays:

```Go
var x = [...]int{10, 20, 30}
var y = [3]int{10, 20, 30}
fmt.Println(x == y) // prints true 
```

Go only has one-dimensional arrays, but you can simulate multidimensional arrays:

```Go
var x[2][3]int
```

This declares `x` to be an array of length 2 whose type is an array of `ints` of length 3. This sounds pedantic, but there are languages with true matrix support; Go isn't one of them.

The built-in function `len` takes in an array and returns its length:

```Go
fmt.Println(len(x))
```

Arrays are rarely used explicitly in Go because they come with an unusual limitation: Go considers the *size* of the array to be part of the *type* of the array. This makes arrays declared to be `[3]int` a different type from an array that's declared to be `[4]int`. This also means you cannot use a variable to specify the size of an array, because types must be resolved at compile time, not at runtime.

Additionally you can't use type conversion to convert arrays of different size to identical types. This means you can't write a function that works with arrays of any size and you can't assign arrays of different sizes to the same variable.

Due to these restrictions you won't use arrays unless you know the exact length you need ahead of time. The main reason why arrays exist in Go is to provide the backing store for *slices*.

## Slices
Most of the time when you want a data structure that holds sequences of values, a slice is what you should use. What makes slices so useful is that the length is *not* part of the type for a slice, removing the limitation that is there for arrays.

Using slices looks very similar to using arrays:

```Go
var x = []int{10, 20, 30}
var x = []int{1, 5: 4, 6, 10: 100, 15}
var x [][]int
```

So far everything seems very similar to arrays, but where we see the differences is when we declare slices without using a literal:

```Go
var x []int
```

Since no value is assigned to `x` it gets assigned the zero value for a slice, which is `nil`. We'll cover `nil` in more detail in [section 6](./6_pointers.md), for now just know that Go uses `nil` as an identifier that represents the lack of a value for some types. `nil` has no type so ti can be assigned or compared against values of different types, and a `nil` slice contains nothing.

A slice is also the first type we've seen that isn't comparable. It is a compile-time error to use `==` or `!=` to compare 2 slices. The only thing you can compare a slice with is `nil`:


