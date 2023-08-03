# Composite Types

<!--toc:start-->
- [Composite Types](#composite-types)
  - [Arrays-Too Rigid to Use Directly](#arrays-too-rigid-to-use-directly)
  - [Slices](#slices)
    - [`len`](#len)
    - [`append`](#append)
    - [Capacity](#capacity)
      - [The Go Runtime](#the-go-runtime)
    - [`make`](#make)
    - [Decalring Your Slice](#decalring-your-slice)
    - [Slicing Slices](#slicing-slices)
      - [Slices share storage sometimes](#slices-share-storage-sometimes)
    - [Converting Arrays to Slices](#converting-arrays-to-slices)
    - [`copy`](#copy)
  - [Strings and Runes and Bytes](#strings-and-runes-and-bytes)
  - [Maps](#maps)
    - [Reading and Writing a Map](#reading-and-writing-a-map)
    - [The comma ok Idiom](#the-comma-ok-idiom)
    - [Deleting from Maps](#deleting-from-maps)
    - [Using Maps as Sets](#using-maps-as-sets)
  - [Structs](#structs)
    - [Anonymous Structs](#anonymous-structs)
    - [Comparing and Converting Structs](#comparing-and-converting-structs)
  - [Wrapping Up](#wrapping-up)
<!--toc:end-->

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

In Go, the size of an array is considered part of the type of the array. This makes an array that's declared `[3]int` a different type than an array decalred to be `[4]int`. This also means that you cannot use a variable to specify the size of an array, because types must be resolved at compile time.

Additionally you can't use a type conversion to convert arrays of different sizes to identical types. Because of this you can't write a function that works with arrays of any size and you can't assign arrays of different sizes to the same variable. 

> We'll learn how arrays work behind the scenes when we cover memory layout in [section 6](./6_pointers.md).

Becauses of these restrictions only use arrays when you know the exact length you need ahead of time. The main reason why arrays exist in Go is to provide the backing store for *slices*, one of the most useful features of Go.

## Slices
What makes slices so useful is that length is not part of the type for a slice. This means we can write a single function that process slices of any size(we'll cover functions is [section 5](./5_functions.md)), and we can grow slices as needed. 

Working with slices is similar to arrays with a few subtle differences. This first thing to notice is that we don't specify the size of the slice when we declare it:

```go 
var x = []int{10, 20, 30}
```

This creates a slice of 3 `int`s using a *slice literal*. Here are a few more examples using slices:

```go 
var x = []int{1, 5: 4, 6, 10: 100, 15} // [1, 0, 0, 0, 0, 4, 6, 0, 0, 0, 100, 15]
var y [][]int // simulating multidimensional slice
x[0] = 10
fmt.Println(x[2])
```

So far everything seem identical to arrays, but we start to see the differences when we look at declaring a slice without using a literal:

```go 
var x []int
```

This creates a slice of `int`s. Since no value is assigned, `x` is assigned the zero value for a slice which is `nil`. In Go, `nil` is an identifier that represents the lack of a value for some types. Like untyped numeric constants, `nil` has not type so it can be assigned or compared against values of different types. A `nil` slice contains nothing.

A slice is the first type we've seen that isn't *comparable*. It is a compile-time error to use `==` or `!=` to compare slices. The only thing you can compare a slice with is `nil`:

```go 
fmt.Println(x == nil)
```

> The `reflect` package contains a function called `DeepEqual` that can compare almost anything, including slices. It's primarily intended for testing but can be used to compare slices if needed. We'll look at it when talking about reflection in [section 14](./14_here_there_be_dragons:reflect_unsafe_and_cgo.md).

### `len`
Go provides several built-in functions to work with its built-in types. We saw the built-in `len` function when looking at arrays and it works for slices too. When you pass a `nil` slice to `len` it returns 0.

> Functions like `len` are built-in to Go because they can do things that can't be done by the functions that you write. `len`'s parameters can be any type of array, any type of slice, strings, maps and even channels. Trying to pass a variable of any other type to `len` is a compile-time error. We'll see in [section 5](./5_functions.md) that Go doesn't let developers write functions that behave this way.

### `append`
The built-in `append` function is used to grow slices:

```go
var x []int
x = append(x, 10)

var y = []int{1, 2, 3}
y = append(y, 4)

y = append(y, 5, 6, 7)
```

The `append` function takes at least two parameters, a slice of any type and a value of that type. It returns a slice of the same type and that slice is assigned back to that slice that's passed in.

You can also append a slice onto another using the `...` operator to expand the source slice into individual values.

```go 
y := []int{20, 30, 40}
x = append(x, y...)
```

It is a compile-time error if you forget to assign the value returned form `append`. We will talk about this further in [section 5](./5_functions.md), but Go is a *call by value* language. Every time you pass a parameter to a function, Go makes a copy of the value that's passed in. Passing a slice to the `append` function actually passes a copy of the slice to the function. The function adds the values to the copy of the slice and returns the copy. You then assign the returned slice back to the variable in the calling function.

### Capacity
Each element in a slice is assigned to consecutive memory locations, making it quick to read or write these values. Every slice has a *capacity*, which is the number of consecutive memory locations reserved. The capacity can be larger than the length. Each time you append to a slice the length is increased by one, when the length reaches the capacity, there's no more room to add values. If you try to add additional values when the length equals the capacity, the `append` function uses the Go runtime to allocate a new slice with a larger capacity. The values in the original slice are copied to the new slice, the new values are added to the end, and the new slice is returned.

> #### The Go Runtime
> The Go runtime provides services like memory allocation and garbage collection, concurrency support, networking, and implementations of built-in types and functions.
>
> The Go runtime is compiled into every Go binary. This is different than other languages that use a virtual machine, which must be installed separately to allow programs in those languages to run. Including the runtime in the binary makes it easier to distribute Go programs and avoid compatibility issues between the runtime and the program.

When a slice grows via `append` it takes time for the Go runtime to allocate new memory and copy the existing date from the old memory to the new. The old memory also needs to be garbage collected.

The built-in`cap` function returns the capacity of a slice. This is most often used to check if a slice is large enough to hold new data, or if a call to `make` is needed to create a new slice.

Let's take a look at how adding elements to a slice changes the length and capacity:

```go 
var x []int
fmt.Println(x, len(x), cap(x))
x = append(x, 10)
fmt.Println(x, len(x), cap(x))
x = append(x, 20)
fmt.Println(x, len(x), cap(x))
x = append(x, 30)
fmt.Println(x, len(x), cap(x))
x = append(x, 40)
fmt.Println(x, len(x), cap(x))
x = append(x, 50)
fmt.Println(x, len(x), cap(x))
```

The follow is the output of the above code, notice how and when the capacity increases.

```shell
[] 0 0
[10] 1 1
[10 20] 2 2
[10 20 30] 3 4
[10 20 30 40] 4 4
[10 20 30 40 50] 5 8
```

While it is nice that slices grow automatically it's far more efficient to size them once. If you know how much space you need create the slice with the correct initial capacity with the `make` function.

### `make`
We've seen how to create a slice using a slice literal or the `nil` zero value but neither of these options allow you to create an empty slice that already has a length or capacity specified. This is the job of the built-in `make` function:

```go 
x := make([]int, 5) // creates an int slice with a length of 5 and a capacity of 5
```

A common beginner mistake is to try and populate the initial elements using `append`:

```go 
x := make([]int, 5)
x = append(x, 10) // [0 0 0 0 0 10], len: 6, cap: 10
```

The 10 is placed at the end of the slice, *after* the zero values in positions 0-4 because `append` always increases the length of a slice. 

We can also specify an initial capacity with `make`:

```go 
x := make([]int, 5, 10)

y := make([]int, 0, 10) // In this case we have a non-nil slice with a length of 0
y = append(y, 5, 6, 7, 8) // Because length is 0 we can't directly index into it but we can append values
fmt.Println(y) // [5 6 7 8]
```

### Decalring Your Slice
The primary goal when declaring a slice is to minimize the number of times the slice needs to grow.

If it's possible that the slice won't need to grow at all(because your function might return nothing), use a `var` declaration with no assigned value to create a `nil` slice.

```go 
var data []int
```

If you have some starting values, or if a slice's values aren't going to change, then a slice literal is a good choice.

```go 
data := []int{2, 4, 6, 8}
```

If you have a good idea of h ow large your slice needs to be, but don't know what those values will be you should use `make`. The question then is, should you specify a nonzero length in the call to `make` or specify a zero length and a nonzero capacity.

- If you are using a slice as a buffer then specify a nonzero length
- If you are *sure* you know the exact size you want, specify the length and index into the slice to set the values. This is often done when transforming values in one slice and storing them in a second. The downside to this approach is that if you have the size wrong, you'll either end up with zero values at the end of the slice or a panic from trying to access elements that don't exist.
- In other situations use `make` with a zero length and capacity specified. With this approach you use `append` to add items to the slice. If the number of items turns out to be smaller you won't have extra zero values at the end and you code won't panic if there are more values than you were expecting.

### Slicing Slices
A *slice expression* creates a slice from a slice. It's written inside brackets and consists of a starting offset and an ending offset, separated by a colon(:). If you leave off the staring offset, 0 is assumed. Likewise, leaving off the ending offset, the end of the slice is assumed.

```go 
x := []int{1, 2, 3, 4}
y := x[:2]   // [1 2]
z := x[1:]   // [2 3 4]
d := x[1:3]  // [2 3]
e := x[:]    // [1 2 3 4]
```

#### Slices share storage sometimes
When you take a slice from a slice, you are *not* making a copy of the data. Instead, you have two variables that are sharing memory. This means changes to an element in a slice affects all slices that share that element:

```go 
x := []int{1, 2, 3, 4}
y := x[:2]
z := x[1:]
x[1] = 20
y[0] = 10
z[1] = 30
fmt.Println(x) // [10 20 30 4]
fmt.Println(y) // [10 20]
fmt.Println(z) // [20 30 4]
```

Slicing slices gets extra confusing when combined with `append`:

```go 
x := make([]int, 0, 5)
x = append(x, 1, 2, 3, 4)
y := x[:2]
z := x[2:]
fmt.Println(cap(x), cap(y), cap(z)) // 5 5 3
y = append(y, 30, 40, 50)
x = append(x, 60)
z = append(z, 70)
fmt.Println("x:", x) // [1 2 30 40 70]
fmt.Println("y:", y) // [1 2 30 40 70]
fmt.Println("z:", z) // [30 40 70]
```

To avoid complicated slice situations, you should either never use `append` with sub-slices or make sure that `append` doesn't cause an overwrite by using a *full slice expression*. The full slice expression includes a third part which indicates the last position in the parent slice's capacity that's available for the sub-slice.

```go 
x := make([]int, 0, 5)
y := x[:2:2]
z := x[2:4:4]
fmt.Println(cap(x), cap(y), cap(z)) // 5 2 2
y = append(y, 30, 40, 50)
x = append(x, 60)
z = append(z, 70)
fmt.Println("x:", x) // [1 2 3 4 60]
fmt.Println("y:", y) // [1 2 30 40 50]
fmt.Println("z:", z) // [3 4 70]
```

Both `y` and `z` have a capacity of 2. Because we limited the capacity of the sub-slices to their lengths, appending additional elements onto `y` and `z` created new slices that didn't interact with the other slices.

### Converting Arrays to Slices
If you have an array, you can take a slice from it using a slice expression, however remember that taking a slice from an array has the same memory-sharing properties as taking a slice from a slice.

```go 
x := [4]int{5, 6, 7, 8}
y := x[:2]
z := x[2:]
x[0] = 10
fmt.Println(x) // [10 6 7 8]
fmt.Println(y) // [10 6]
fmt.Println(z) // [7 8]
```

### `copy`
To create a slice that's independent of the original use the built-in `copy` function:

```go 
x := []int{1, 2, 3, 4}
y := make([]int, 4)
num := copy(y, x)
fmt.Println(y, num) // [1 2 3 4] 4
```

The `copy` function takes two parameters, the destination slice and the source slice. It copies as many values as it can from source to destination, limited by whichever slice is smaller and returns the number of elements copied.

You can also copy from the middle of the source slice:

```go 
x := []int{1, 2, 3, 4}
y := make([]int, 2)
copy(y, x[2:])
```

The `copy` function allows you to copy between two slices that cover overlapping sections of an underlying slice:

```go 
x := []int{1, 2, 3, 4}
num = copy(x[:3], x[1:])
fmt.Println(x, num) // [2 3 4 4] 3
```

You can use `copy` with arrays by taking a slice of the array. You can make the array either the source or the destination of the copy:

```go 
x := []int{1, 2, 3, 4}
d := [4]int{5, 6, 7, 8}
y := make([]int, 2)
copy(y, d[:])
fmt.Println(y) // [5 6]
copy(d[:], x)
fmt.Println(d) // [1 2 3 4]
```

## Strings and Runes and Bytes
Strings in Go may seem like they are made up of runes, but that's not the case. Go uses a sequence of bytes to represent a string, they don't have to be in any particular character encoding but several Go library functions(and the `for - range` loop) assume that a string is composed of a sequence of UTF-8-encoded code points.

Similarly to how you can extract a single value from an array or slice, you can extract a single value from a string using an index expression:

```go 
var s string = "Hello there"
var b byte = s[6]       // t 
var s2 string = s[4:7]  // o t
var s3 string = s[:5]   // Hello
var s4 string = s[6:]   // there
```

While it can be handy to use index notation to extract individual elements we have to be careful when doing so. A string is composed of a sequence of bytes, while a code point in UTF-8 can be anywhere from one to four bytes long. When dealing with languages other than English or with emojis/symbols you will run into code points that are multiple bytes long in UTF-8:

```go 
var s string := "Hello 🌞"
var s2 string = s[4:7] // o �
var s3 string = s[:5]  // Hello 
var s4 string = s[6:]  // 🌞 
```

In the above example `s2` is set to "o �" because we only copied the first byte of the sun emoji's code point, which in invalid.

Go allows you to pass a string to the built-in `len` function to find the length of the string, but note that the length returned is the length in bytes, not in code points:

```go 
var s string := "Hello 🌞"
fmt.Println(len(s)) // Prints 10 not 7
```

> Even though Go allows you to use slicing and indexing syntax with strings, you should only use it when you know that your string only contains characters that take up one byte.

Because of the relationship between runes, strings, and bytes, Go has some interesting type conversions between these types. A single rune or byte can be converted to a string:

```go 
var a rune = 'x'
var s string = string(a)
var b byte = 'y'
var s2 string = string(b)
```

> A common bug for new Go developers is to try to make an `int` into a `string` using type conversion:
> ```go
> var x int = 65
> var y = string(x)
> fmt.Println(y) // This prints "A" not "65"
> ```

A string can be converted back and forth to a slice of bytes or a slice of runes:

```go 
var s string = "Hello, 🌞"
var bs []byte = []byte(s)
var rs []rune = []rune(s)
fmt.Println(bs) // [72 101 108 108 111 44 32 240 159 140 158]
fmt.Println(rs) // [72 101 108 108 111 44 32 127774]
```

Most data in Go is read and written as a sequence of bytes, so the most common string type conversions are back and forth with a slice of bytes.

Instead of using slice and index expressions with strings, you should extract substrings and code points from strings using the functions in the `strings` and `unicode/utf8` packages in the standard library.

## Maps
The map type is written as `map[keyType]valueType` and there are a few different ways that you can declare a map. First, you can use a `var` declaration to create a map variable that's set to its zero value:

```go 
var nilMap map[string]int
```

In this case, `nilMap` is declared to be a map with `string` keys and `int` values. The zero value for a map is `nil`. A `nil` map has a length of 0 and attempting to read from a `nil` map always returns the zero value for the map's value type. Attempting to write to a `nil` map causes a panic.

We can use a `:=` declaration to create a map variable by assigning it a *map literal*:

```go 
totalWins := map[string]int{}
```

Here we used an empty map literal which isn't the same as a `nil` map. It has a length of 0, but you can read and write to a map assigned an empty map literal. Here is a map with a nonempty map literal:

```go 
teams := map[string][]string {
    "Orcas": []string{"Fred", "Ralph", "Bijou"},
    "Lions": []string{"Sarah", "Peter", "Billie"},
    "Kittens": []string{"Waldo", "Raul", "Ze"},
}
```

In this example, the value is a slice of strings. The type of the value in a map can be anything, though there are some restrictions on the types of the keys.

If you know how many key-value pairs you intend to put in the map, but don't know the exact values you can use `make` to create a map with a default size:

```go 
ages := make(map[int][]string, 10)
```

Maps created with `make` still have a length of 0 and they can grow past the initially specified size.

Maps are like slices in several ways:
- Maps automatically grow as you add key-value pairs to them.
- If you know how many key-value pairs you plan to insert into a map, you can use `make` to create a map with a specific initial size.
- Passing a map to the `len` function tells you the number of key-value pairs in the map.
- The zero value for a map is `nil`.
- Maps are not comparable. You can check if they are equal to `nil` but you can't compare maps using `==` or `!=`.

The key for a map must be a comparable type, this means that you cannot use a slice or a map as they key for a map.

> Use a map when the order of elements doesn't matter and a slice when the order of elements is important.

### Reading and Writing a Map
```go 
totalWins := map[string]int{}
totalWins["Orcas"] = 1
totalWins["Lions"] = 2
fmt.Println(totalWins["Orcas"])     // 1
fmt.Println(totalWins["Kittens"])   // 0
totalWins["Kittens"]++
fmt.Println(totalWins["Kittens"])   // 1
totalWins["Lions"] = 3
fmt.Println(totalWins["Lions"])     // 3
```

We assign a value to a map key by putting the key within brackets and using `=` to specify the value. We read the value assigned to a map key by putting the key within brackets. Note that you cannot use `:=` to assign a value to a map key.

When trying to read the value assigned to a map key that was never set, the map returns the zero value for the map's value type.

### The comma ok Idiom
We've seen how a map returns the zero value if you ask for a value associated with a key that's not in the map. This can be useful when implementing things like a counter but there are time where you need to find out if a key is in a map. To do this Go provides the *comma ok idiom* to tell the difference between a key that's associated with a zero value and key that's not in the map.

```go 
m := map[string]int{
    "hello": 5,
    "world": 0,
}
v, ok := m["hello"]
fmt.Println(v, ok) // 5 true

v, ok = m["world"]
fmt.Println(v, ok) // 0 true

v, ok = m["goodbye"]
fmt.Println(v, ok) // 0 false
```

With the comma ok idiom you assign the results of a map read to two variables. The first gets the value associated with the key, the second value returned is a bool and it is usually named `ok`. If `ok` is `true` the key is present in the map, if `ok` is `false` they key is not present.

### Deleting from Maps
Key-value pairs are removed from a map via the built-in `delete` function:

```go 
m := map[string]int{
    "hello": 5,
    "world": 0,
}
delete(m, "hello")
```

The `delete` function takes a map and a key and then removes the key-value pair with the specified key. If the key isn't present or the map is `nil` nothing happens. The `delete` function doesn't return a value.

### Using Maps as Sets
A set is a data type that ensures there is at most one of a value but doesn't guarantee a particular order. Checking to see if an element is in a set is fast, no matter how many elements are in the set. 

Go doesn't include a set, but you can use a map to simulate some of its features. Set the key of the map to the type that you want to put into the set and use a `bool` for the value:

```go 
intSet := map[int]bool{}
vals := []int{5, 10, 2, 5, 8, 7, 3, 9, 1, 2, 10}
for _, v := range vals {
    intSet[v] = true
}
fmt.Println(len(vals), len(intSet)) // 11 8
fmt.Println(intSet[5])              // true
fmt.Println(intSet[500])            // false
if intSet[100] {
    fmt.Println("100 is in the set") // Doesn't print 
}
```

If your set needs operations like union, intersection, or subtraction you can write your or use a third-party library that provides that functionality(More on third-party libraries in [section 9](./9_modules_packages_imports.md))

Some people prefer to use `struct{}` for the value when a map is being used to implement a set. The advantage being that an empty struct uses zero bytes, while a boolean uses one byte. However, using a `struct{}` makes your code less readable and you need to use the comma ok idiom to check if a value is in the set:

```go 
intSet := map[int]struct{}{}
vals := []int{5, 10, 2, 5, 8, 7, 3, 9, 1, 2, 10}
for _, v := range vals {
    intSet[v] = struct{}{}
}
if _, ok := intSet[5]; ok {
    fmt.Println("5 is in the set")
}
```

Unless you have very large sets, it is unlikely that the difference in memory usage is significant enough to outweigh the disadvantages.

## Structs
Most languages have a concept that's similar to a struct, and the syntax that Go uses to read and write structs should look familiar:

```go 
type person struct {
    name string
    age int 
    pet string
}
```

A struct type is defined with keyword `type`, the name of the struct type, they keyword `struct`, and a pair of braces({}). Within the braces, you list the fields in the struct. Note that unlike map literals, there are no commas separating the fields in a struct declaration. Struct types can be defined inside or outside of a function, those defined within a function can only be used within that function.

Once a struct type is declared, we can define variables of that type:

```go 
var fred person
```

Since no value is assigned to `fred`, it gets the zero value for the `person` struct type. A zero value struct has every field set to the field's zero value.

You can also assign a *struct literal* to a variable:

```go 
bob := person{}
```

Unlike maps, there is no difference between assigning an empty struct literal and not assigning a value at all. Both initialize all of the fields in the struct to their zero values. 

There are two different style for a nonempty struct literal:

```go 
julia := person{
    "Julia",
    40,
    "cat",
}

```

The first format uses a comma-separated list of values, when using this format a value for every field in the struct must be specified and values are assigned to the fields in the order they were declared in the struct definition.

```go 
beth := person {
    age: 30,
    name: "Beth",
}
```

The second format uses the names of the fields in the struct to specify the values. When you use this style, you can leave out keys and specify fields in any order.

A field in a struct is accessed with dotted notation:

```go 
bob.name = "Bob"
fmt.Println(beth.name)
```

### Anonymous Structs
You can also declare that a variable implements a struct type without first giving the struct type a name, this is known as an *anonymous struct*:

```go 
var person struct {
    name string 
    age int 
    pet string
}

person.name = "bob"
person.age = 50
person.pet = "dog"

pet := struct {
    name string 
    kind string
}{
    name: "Fido",
    kind: "dog",
}
```

You might wonder when it's useful to have a data type that's only associated with a single instance. There are two common situations where anonymous structs come in handy. The first is when you translate external data into a struct or a struct into external data(like JSON or protocol buffers). This is called *unmarshaling* and *marshaling* data. 

Writing tests is another place where anonymous structs pop up. We'll use a slice of anonymous structs when writing table-driven tests in [section 13](./13_writing_tests.md)

### Comparing and Converting Structs
Whether or not a struct is comparable depends on the struct's fields. Structs that are composed entirely of compatible types are comparable; those with slice, map, or channel fields are not.

Just like how Go doesn't allow comparisons between variables of different primitive types, Go doesn't allow comparisons between variables that represent structs of different types. Go does allow a type conversion from one struct type to another *if the fields of both structs have the same names, order, and types*. 

If two struct variables are being compared and at least one of them has a type that's an anonymous struct, you can compare them without a type conversion if the fields of both structs have the same names, order, and types. You can also assign between named and anonymous struct types if they are comparable.

```go 
type firstPerson struct {
    name string 
    age int
}
f := firstPerson{
    name: "Bob",
    age: 50,
}

var g struct{
    name string
    age int
}

g = f 
fmt.Println(f == g)
```

## Wrapping Up
This section covered some additional information about strings, and the use of the built-in generic container types, slice and maps. We also looked at constructing our own composite types via structs. In the next [section](./4_blocks_shadows_control_structures.md) we will look at Go's control structures(`for`, `if/else`, and `switch`) as well as how Go organizes code into blocks and how block levels can affect code execution.
