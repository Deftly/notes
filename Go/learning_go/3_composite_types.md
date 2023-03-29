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

```Go
fmt.Println(x == nil) // prints true
```

### len
Go provides several built-in functions to work with its built-in types. We've seen `len` previously with arrays and it works for slices too.

> **_NOTE:_** Functions like `len` are built in to Go because they do things that can't be done by functions that we write. `len`'s parameter can be any type of array or any type of slice. We'll also see that it works for strings, maps, and channels too. When we look at functions in [section 5](./5_functions.md) we'll see that Go doesn't let developers write functions that behave this way.

### append
the built-in `append` is used to grow slices:

```Go
var x []int
x = append(x, 10)
```

The `append` function takes at least two parameters, a slice of any type and a value of that type. It returns a slice of the same type. The returned slice is assigned back to the slice that's passed in. 

```Go
var x = []int{1, 2, 3}
x = append(x, 4)

x = append(x, 5, 6, 7) // Can append more than one value at a time

y = []int{20, 30, 40}
x = append(x, y...) // One slice is appened onto another using the ... operator
```

It is a compile time error if your forget to assign the value returned from `append`. This may seem a bit repetitive, and we'll talk about this in greater detail in [section 5](./5_functions.md), but Go is a *call by value* language. Every time you pass a parameter to a function Go makes a copy of the value that's passed in. Passing a slice to the `append` function actually passes a copy of the slice to the function. The function adds the values to the copy and returns the copy. You then assign the returned slice back to the variable in the calling function. 

### Capacity
Each element in a slice is assigned to consecutive memory locations which makes it quick to read and write values. Every slice has a *capacity* which is the number of consecutive memory location reserved. The capacity can be larger than the length. Each time you append to a slice you increase the length, when the length reaches the capacity there is no more room to put values. If you try to add more values than there is capacity, the `append` function uses the Go runtime to allocate a new slice with a larger capacity, copying the values in the original slice to the new slice.

> **_NOTE:_** The Go runtime provides services like memory allocation and garbage collection, concurrency support, networking, and implementations of built-in types and functions. The Go runtime is compiled into every Go binary. This is different from languages that use a virtual machine, which must be installed separately to allow programs written in those languages to function. Including the runtime in the binary makes it easier to distribute Go programs and avoid worries about compatibility issues between the runtime and the program.

The built-in `cap` function returns the capacity of a slice. This is usually used to check if a slice is large enough to hold new data, or if a call to `make` is needed to create a new slice.

```Go
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

Here's the output of the above code, pay attention to how and when the capacity increases:

```Shell
[] 0 0
[10] 1 1
[10 20] 2 2
[10 20 30] 3 4
[10 20 30 40] 4 4
[10 20 30 40 50] 5 8
```

While it is nice that slices grow automatically it's far more efficient size them once. If you know how many things you plan to put into a slice, create it with the correct initial capacity using the `make` function.

### make
While creating a slice with a literal or `nil` is useful neither option lets us create an empty slice that already has a length and capacity specified. To do so we use the built-in `make` function:

```Go
x := make([]int, 5)
```

This creates an `int` slice with a length and capacity of 5, all the elements of which are initialized to 0.

We can also specify an initial capacity with `make`:

```Go
x := make([]int, 5, 10)
y := make([]int, 0, 10)
```

> **_NOTE:_** Never specify a capacity that is less than the length! It is a compile time error to do so with a constant or numeric literal and a runtime panic if done with a variable.

### Declaring Your Slice
The primary goal when choosing how to declare your slice is to minimize the number of times the slice needs to grow. 

If it's possible that your slice won't need to grow at all (because your function might return nothing), use a `var` declaration with no assigned value to create a `nil` slice.

If you have some starting values, or if a slice's values aren't going to change then a slice literal is a good choice.

If you have a good idea of how large your slice needs to be, but don't know what those values will be when you are writing the program, use `make`. Then the question is whether to specify a nonzero length or zero length and a nonzero capacity:
- If you are using a slice as a buffer, use a nonzero length.
- If you are *sure* that you know the exact size, you can specify the length and index into the slice to set values. This is often done for transformations, the downside is that if you have the size wrong you'll either end up with extra zero values at the end of the slice or a panic from trying to access elements that don't exist.
- In other situations, use `make` with a zero length and specify the capacity.

### Slicing Slices
A *slice expression* creates a slice from a slice. It is written inside brackets and consists of a starting offset and ending offset, separated by a colon(:).

```Go
x := []int{1, 2, 3, 4}
y := x[:2]  // [1 2]
z := x[1:]  // [2 3 4]
d := x[1:3] // [2 3]
e := x[:]   // [1 2 3 4]
```

#### Slice share storage sometimes
When you take a slice from a slice, you are *not* making a copy of the data. Instead, you now have two variables that are sharing memory. This means that changes to an element in a slice affect all slices that share that element.

```Go
x := []int{1, 2, 3 4}
y := x[:2]
z := x[1:]
x[1] = 20
y[0] = 10
z[1] = 30
fmt.Println("x:", x) // x: [10 20 30 4]
fmt.Println("y:", y) // y: [10 20]
fmt.Println("z:", z) // z: [20 30 4]
```

The following scenario shows multiple slices appending and overwriting each other's data:

```Go
x := make([]int, 0, 5)
x = append(x, 1, 2, 3, 4)
y := x[:2]
z := x[2:]
fmt.Println(cap(x), cap(y), cap(z))
y = append(y, 30, 40, 50)
x = append(x, 60)
z = append(z, 70)
fmt.Println("x:", x) // [1 2 30 40 70]
fmt.Println("y:", y) // [1 2 30 40 70]
fmt.Println("z:", z) // [30 40 70]
```

To avoid complicated situations, you should either never use `append` with a sub-slice or make sure that `append` doesn't cause an overwrite by using a *full slice expression*. This makes it clear how much memory is shared between the parent slice and the sub-slice by adding a third part which indicates the last position in the parent slice's capacity that's available for the sub-slice.

```Go
y := x[:2:2]
z := x[2:4:4]
```

### Converting Arrays to Slices
If you have an array you can take a slice from it using a slice expression. However, be aware that taking a slice from an array has the same memory-sharing properties as taking a slice from a slice. 

```Go
x := [4]int{5, 6, 7, 8}
y := x[:2]
z := x[2:]
x[0] = 10
fmt.Println("x:", x) // [10 6 7 8]
fmt.Println("y:", y) // [10 6]
fmt.Println("z:", z) // [7 8]
```

### copy
If you need to create a slice that's independent of the original, use the built-in `copy` function:

```Go
x := []int{1, 2, 3, 4}
y := make([]int, 4)
num := copy(y, x)
fmt.Println(y, num) // [1 2 3 4] 4
```

The `copy` function takes 2 parameters. The first is the destination slice and the second is the source slice. It copies as many values as it can from source to destination, limited by whichever slice is smaller, and returns the number of elements copied. 

Some more ways that you can use `copy`:

```Go
x := []int{1, 2, 3, 4}
y := make([]int, 2)
num = copy(y, x)

a := []int{1, 2, 3, 4}
b := make([]int, 2)
copy(b, a[2:])
```

## Strings and Runes and Bytes
You might think that a string in Go is made out of runes, but that's not the case. Go uses a sequence of bytes to represent a string. These bytes don't have to be in any particular character encoding but several Go library functions assume that a string is composed of a sequence of UTF-8 encoded code points.

Just like an array or a slice, you can use index expression with strings as well as slice expression notation:

```Go
var x string = "Hello there"
var b byte = s[6] // "t"

var s2 string = s[4:7] // "o t"
var s3 string = s[:5]  // "Hello"
var s4 string = s[6:]  // "there"
```

While it can be handy that Go allows the use of these notations to make substrings and extract individual entries from a string we need to be careful when doing so. Strings are immutable so they don't have any modification problems but there is another problem. A string is composed of a sequence of bytes, while a code point in UTF-8 can be anywhere from one to four bytes long. The previous example was composed entirely of code points that are one byte long in UTF-8 but that won't always be the case. Code points that take more than one byte can lead to errors if we extract only some of the bytes needed for a code point.

Because of the complicated relationship between runes, strings, and bytes, Go has some interesting type conversions between these types. A single rune or byte can be converted to a string:

```Go
var a rune = 'x'
var s string = string(a)
var b byte = 'y'
var s2 string = string(b)

// Common bug for new Go developers
var x int = 65
var y = string(x)
fmt.Println(y) // y = "A" not "65"
```

A string can be converted back and forth to a slice of bytes or a slice or runes:

```Go
var s string = "Hello, 🌞"
var bs []byte = []byte(s)
var rs []rune = []rune(s)

fmt.Println(bs) // [72 101 108 108 111 44 32 240 159 140 158]
fmt.Println(rs) // [72 101 108 108 111 44 32 127774]
```

Most data in Go is read and written as a sequence of bytes, so the most common string type conversions are back and forth with a slice of bytes. Slices of runes are uncommon.

Rather than use slice and index expressions with strings, you should extract sub-strings and code points from strings using the functions in the `strings` and `unicode/utf8` packages in the standard library. In the next [section](./4_blocks_shadows_control_structures.md), we'll see how to use a `for-range` loop to iterate over the code points in a string.

## Maps
The map type is written as `map[keyType]valueType`. Let's take a look at a few ways to declare maps. First you can use a `var` declaration to create a map variable that's set to it's zero value:

```Go
var nilMap map[string]int
```

In this case, `nilMap` is declared to be a map with `string` keys and `int` values. The zero value for a map is `nil` and has a length of 0. Attempting to read a `nil` map always returns the zero value for the map's value type, while attempting to write to a `nil` map variable causes a panic.

We can use a `:=` declaration to create a map variable by assigning it a map literal:

```Go
totalWins := map[string]int{}
```

In this case we are using an empty map literal. This is not the same as a `nil` map. It has a length of 0 but you can read and write to it. Here's what a nonempty map literal looks like:

```Go
teams := map[string][]string {
  "Orcas": []string{"Fed", "Ralph", "Bijou"},
  "Lions": []string{"Sarah", "Peter", "Billie"},
  "Kittens": []string{"Waldo", "Raul", "Ze"},
}
```

If you know how many key-value pairs you intend to put in the map, but don't know the exact values, you can use `make` to create a map with a default size:

```Go
ages := make(map[int][]string, 10)
```

Maps created with `make` still have a length of 0, and they can grow past the initially specified size.

Maps are like slices in several ways:
- Maps automatically grow as you add key-value pairs to them.
- If you know how many key-value pairs you plan to insert you can use make to specify the initial size.
- Passing a map to `len` returns the number of key-value pairs in a map.
- The zero value for a map is `nil`.
- Maps are not comparable. You can check if they are equal to `nil`, but you cannot check if two maps have identical keys and values using `==` or differ using `!=`.

The key for a map can be any comparable type, this means you cannot use a slice or map as the key for a map.

> **_NOTE:_** Use a map when the order of elements doesn't matter. Use a slice when ordering is important.

### Reading and Writing a Map

```Go
totalWins := map[string]int{}
totalWins["Orcas"] = 1
totalWins["Lions"] = 2
fmt.Println(totalWins["Orcas"])   // 1
fmt.Println(totalWins["Kittens"]) // 0
totalWins["Kittens"]++
fmt.Println(totalWins["Kittens"]) // 1
totalWins["Lions"] = 3
fmt.Println(totalWins["Lions"])   // 3
```

### The comma ok Idiom
As we've seen, a map returns the zero value if you ask for the value associated with a key that's not in the map. This is handy when implementing things like a counter. However, there are time when you need to find out if a key is in the map. Go provides the *comma ok idiom* to tell the difference between a key that's associated with a zero value and a key that's not in the map:

```Go
m := map[string]int{
  "hello": 5,
  "world": 0,
}

v, ok := m["hello"]
fmt.Println(v, ok)   // 5 true

v, ok = m["world"]
fmt.Println(v, ok)   // 0 true 

v, ok = m["goodbye"] // 0 false
fmt.Println(v, ok)
```

The comma ok idiom assigns the results of a map read to two variables. The first gets the value associated with the key. The second value is a bool, which is usually named `ok`, which if `true` means the key is present and if `false` means the key is not present.

### Deleting from Maps
Key-value pairs are removed from a map via the built-in `delete` function:

```Go
m := map[string]int{
  "hello": 5,
  "world": 10,
}
delete(m, "hello")
```

The `delete` function takes a map and key and then removes the key-value pair with the specified key. If the key isn't present or the map is `nil`, nothing happens.

### Using Maps as Sets
Go doesn't include a set, but you can use a map to simulate some of its features. Use the key of the map for the type that you want to put into the set and use a `bool` for the value:

```Go
intSet := map[int]bool{}
vals := []int{5, 10, 2, 5, 8, 7, 3, 9, 1, 2, 10}
for _, v := range vals {
  intSet[v] = true
}
fmt.Println(len(vals), len(intSet)) // 11 8
fmt.Println(intSet[5])              // true
fmt.Println(intSet[500])            // false
if intSet[100] {
  fmt.Println("100 is in the set")
}
```

For sets that provide operations like union, intersection, and subtraction, you can either write one yourself or use one of the many third-party libraries that provide the functionality(We'll go over how to use third-party libraries in [section 9](./9_modules_packages_imports.md).

Some people prefer to use `struct{}` for the value when a map is being used to implement a set(We'll cover structs right after this). The advantage is that an empty struct uses 0 bytes, while a boolean uses one byte. The downside is your code is more clumsy, you have less obvious assignment, and you need to use the comma ok idiom to check if a value is in the set:

```Go
intSet = map[int]struct{}{}
vals := []int{5, 10, 2, 5, 8, 7, 3, 9, 1, 2, 10}
for _, v := range vals {
  intSet[v] := struct{}{}
}
if _, ok := intSet[5]; ok {
  fmt.Println("5 is in the set")
}
```

Unless you have very large sets, it is unlikely that the difference in memory usage is significant enough for you to want to do this.

## Structs
Maps are convenient for storing some kinds of data but they have limitations. They don't define an API since there's no way to constrain a map to only allow certain keys. Also, all of the values in a map must be of the same type. For these reasons, maps aren't ideal for passing data from function to function. Instead, when you have related data that you want grouped together, you should define a *struct*.

```Go
type person struct {
	name string
	age  int
	pet  string
}
```

A struct type is defined by the keyword `type`, the name of the struct type, the keyword `struct`, and a pair of braces({}). Within the braces you list the fields in the struct. You can define a struct type inside or outside of a function, though structs defined in a function can only be used within that function(More on functions in [section 5](./5_functions.md)). 

Once a struct type is declared we can define variables of that type:

```Go
var fred person // var declaration
bob := person{} // struct literal declaration
```

Unlike maps, there is no difference between assigning an empty struct literal and not assigning a value at all. Both initialize fields in the struct to their zero values. 

A field in a struct is accessed with dotted notation:

```Go
bob.name = "Bob"
fmt.Println(beth.name)
```

### Anonymous Structs
You can also declare that a variable implements a struct type without first giving the struct type a name. This is called an *anonymous struct*:

```Go
var person struct {
  name string
  age  int
  pet  string
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

In the above example, the types of the variables `person` and `pet` are anonymous structs. There are 2 common situations where anonymous structs are useful. The first is when translating external data into a struct or a struct into external data(like JSON or protocol buffers). This is called *unmarshaling* and *marshaling*  data. Writing tests is another place where they are common. We'll use a slice of anonymous structs when writing table-driven tests in [section 13](./13_writing_tests.md).

### Comparing and Converting Structs
Whether or not a struct is comparable depends on the struct's fields. If it is entirely composed of comparable types then it is comparable, but if it has a slice, map, or channel field then it is not. 

Just as Go doesn't allow comparisons between variables of different types, Go doesn't allow comparisons between variables that represent structs of different types. It does allow conversion from one struct type to another *if the fields of both structs have the same names, order, and types*. 

If one of the two struct variables being compared has an anonymous struct type you can compare them without a type conversion *if both structs have the fields of both structs have the same names, order, and types*. You can also assign between named and anonymous structs if the previous conditions are true.

```Go
type firstPerson struct {
  name string
  age int
}

f := firstPerson{
  name: "Bob",
  age: 50,
}

var g struct {
  name string
  age int
}

g = f
fmt.Println(f == g)
```

## Wrapping Up
In this section we covered more about strings, and how to use the built-in generic container types, slices, and maps. We also looked at how to build our own composite types via structs. In the next [section](./4_blocks_shadows_control_structures.md) we'll look at Go's control structures, `for`, `if/else`, and `switch`. We'll also learn how Go organizes code into blocks, and how the different block levels can lead to surprising behavior. 




