# Types, Methods, and Interfaces

<!--toc:start-->
- [Types, Methods, and Interfaces](#types-methods-and-interfaces)
  - [Types in Go](#types-in-go)
  - [Methods](#methods)
    - [Pointer Receivers and Value Receivers](#pointer-receivers-and-value-receivers)
  - [Use Embedding for Composition](#use-embedding-for-composition)
  - [Embedding is Not Inheritance](#embedding-is-not-inheritance)
  - [A Quick Lesson on Interfaces](#a-quick-lesson-on-interfaces)
  - [Interfaces Are Type-Safe Duck Typing](#interfaces-are-type-safe-duck-typing)
  - [Embedding and Interfaces](#embedding-and-interfaces)
  - [Accept Interfaces, Return Structs](#accept-interfaces-return-structs)
  - [Interfaces and `nil`](#interfaces-and-nil)
  - [The Empty Interface Says Nothing](#the-empty-interface-says-nothing)
  - [Type Assertions and Type Switches](#type-assertions-and-type-switches)
  - [Use Type Assertions and Type Switches Sparingly](#use-type-assertions-and-type-switches-sparingly)
  - [Function Types Are a Bridge to Interfaces](#function-types-are-a-bridge-to-interfaces)
  - [Implicit Interfaces Make Dependency Injection Easier](#implicit-interfaces-make-dependency-injection-easier)
  - [Wire](#wire)
  - [Go Isn't Particularly Object-Oriented(and That's Great)](#go-isnt-particularly-object-orientedand-thats-great)
  - [Wrapping Up](#wrapping-up)
<!--toc:end-->

## Types in Go
```go
type Person struct {
  FirstName string
  LastName string
  Age int
}
```
This should be read a declaring a user-defined type with the name `Person` to have the *underlying type* of the struct literal that follows. You can declare a type at any block level, from the package block down. However, you can only access the type from within its scope. The only exceptions are exported package block level types.

## Methods
The methods for a type are defined at the package block level:
```go
type Person struct {
  FirstName string
  LastName string
  Age int
}

func (p Person) String() string {
  return fmt.Sprintf("%s %s, age %d", p.FirstName, p.LastName, p.Age)
}
```
Method declarations look just like function declarations with one addition: the *receiver* specification. By convention the receiver name is a short abbreviation of the type's name, usually the first letter. Don't use `this` or `self` as names.

Just like functions, methods names cannot be overloaded. You can use the same method names for different types, but you can't use the same method name for two different methods on the same type.

We'll cover packages in a later section, for now just know that methods must be declared in the same package as their associated type. Go doesn't allow you to add methods to types you don't control.

Method invocations look like this:
```go
p := Person {
  FirstName: "Fred",
  LastName: "Fredson",
  Age: 52, }
output := p.String()
```
### Pointer Receivers and Value Receivers
We saw earlier that parameters of pointer type indicate a parameter might be modified by a function. The same rule applies for method receivers too. They can be *pointer receivers*(the type is a pointer) or *value receivers*(the type is a value type).
- If you method modifies the receiver, you *must* use a pointer receiver.
- If your method needs to handle `nil` instances, then it *must* use a pointer receiver.
- If you method doesn't modify the receiver, you *can* use a value receiver.

Whether or not you use a value receiver for a method that doesn't modify the receiver depends on the other methods declared on the type. When a type has *any* pointer receiver methods, a common practice is to be consistent and use pointer receivers for *all* methods.
```go
type Counter struct {
  total int
  lastUpdated time.Time
}

func (c *Counter) Increment() {
  c.total++
  c.lastUpdated = time.Now()
}

func (c Counter) String() string {
  return fmt.Sprintf("total: %d, last updated: %v", c.total, c.lastUpdated)
}

func main() {
  var c Counter
  fmt.Println(c.String()) // total: 0, last updated: 0001-01-01 00:00:00 +0000 UTC
  c.Increment()
  fmt.Println(c.String()) // total: 1, last updated: 2023-10-17 21:51:29.705922122 -0700 PDT m=+0.000062631
}
```
You might have noticed that we were able to call the pointer receiver method even though `c` is a value type. When you use a pointer receiver with a local variable that's a value type, Go automatically converts it to a pointer type: `c.Increment()` becomes `(&c).Increment()`.

Remember that the rules for passing values to functions still apply. If you pass a value type to a function and call a pointer receiver method on the passed value, you are invoking the method on a *copy*:
```go
func doUpdateWrong(c Counter) {
  c.Increment()
  fmt.Println("in doUpdateWrong:", c.String())
}

func doUpdateRight(c *Counter) {
  c.Increment()
  fmt.Println("in doUpdateRight:", c.String())
}

func main() {
  var c Counter
  doUpdateWrong(c)
  fmt.Println("in main:", c.String())
  doUpdateRight(&c)
  fmt.Println("in main:", c.String())
}
```
This produces the following output:
```
in doUpdateWrong: total: 1, last updated: 2023-10-18 08:18:41.394711849 -0700 PDT m=+0.000020651
in main: total: 0, last updated: 0001-01-01 00:00:00 +0000 UTC
in doUpdateRight: total: 1, last updated: 2023-10-18 08:18:41.39483867 -0700 PDT m=+0.000147482
in main: total: 1, last updated: 2023-10-18 08:18:41.39483867 -0700 PDT m=+0.000147482
```
The parameter in `doUpdateRight` is of type `*Counter`, which is a pointer instance but we are able to call both `Increment` and `String` on it. This is because Go considers both pointer and value receivers to be in the *method set* for a pointer instance. For a value instance, only the value receiver methods are in the method set.

### Code Your Methods for `nil` Instances
If you call a method on a `nil` instance which a value receiver you'll get a panic as there is no value being pointed to by the pointer. If it's a method with a pointer receiver, it can work if the method is written to handle `nil` instances. In some cases this can make the code simpler like in this implementation of a binary tree:
```go
type IntTree struct {
	left, right *IntTree
	val         int
}

func (it *IntTree) Insert(val int) *IntTree {
	switch {
	case it == nil:
		return &IntTree{val: val}
	case val < it.val:
		it.left = it.left.Insert(val)
	case val > it.val:
		it.right = it.right.Insert(val)
	}
	return it
}

func (it *IntTree) Contains(val int) bool {
	switch {
	case it == nil:
		return false
	case val < it.val:
		return it.left.Contains(val)
	case val > it.val:
		return it.right.Contains(val)
	default:
		return true
	}
}

func main() {
	var it *IntTree
	it = it.Insert(5)
	it = it.Insert(3)
	it = it.Insert(10)
	it = it.Insert(2)
	fmt.Println(it.Contains(2)) // true
	fmt.Println(it.Contains(12)) // false
}
```
Pointer receivers work just like pointer function parameters, it's a copy of the pointer that's passed into the method. Just like `nil` parameters passed to functions, if you change the copy of the pointer, you haven't changed the original. This means you can't write a pointer receiver method that handles `nil` and makes the original pointer non-nil. If your method can't work for a `nil` receiver, check for `nil` and return an error.

### Methods Are Functions Too
Methods can be used as replacement for functions any time there's a variable or parameter of a function type:
```go
type Adder struct {
	start int
}

func (a Adder) AddTo(val int) int {
	return a.start + val
}

func main() {
	myAdder := Adder{start: 10}
	fmt.Println(myAdder.AddTo(5)) // 15

	// method value
	f1 := myAdder.AddTo
	fmt.Println(f1(10)) // 20

	// method expression
	f2 := Adder.AddTo
	fmt.Println(f2(myAdder, 15)) // 25
}
```
Method values and method expressions aren't just clever corner cases, we'll see a use case for them when looking at dependency injection later in this section.

### Functions Versus Methods
Since you can use methods as a function, how do you know when to declare a function and when you should use a method?

Anytime your logic depends on values that are configured at startup or changed while your program is running, those values should be stored in a struct and that logic should be implemented as a method. If your logic only depends on the input parameters, then it should be a function.

### Type Declarations Aren't Inheritance
In addition to declaring types based on built-in Go types and struct literals, you can also declare a user-defined type based on another user-defined type:
```go
type HighScore Score
type Exployee Person
```
Declaring a type based on another type looks like inheritance but it isn't. The two types have the same underlying type but that's all. There is no hierarchy between the types. In Go you can't assign an instance of type `HighScore` to a variable of type `Score` or vice versa without a type conversion, nor can you assign either of them to a variable of type `int` without a type conversion. Also, any methods defined on `Score` aren't defined on `HighScore`:
```go
// assigning untyped constants is valid
var i int = 200
var s Score = 100
var hs HighScore = 300
hs = s // compilation error
s = i // compilation error
s = Score(i) // ok
hs = HighScore(s) // ok
```
> **_NOTE:_** A type conversion between types that share an underlying type keeps the same underlying storage but associates different methods.

### Types Are Executable Documentation
It's understood that you should declare a struct type to hold a set of related data, but it's less clear when you should declare a user-defined type based on other built-in types. The short answer is that types are Documentation. They make code clearer by providing a name for a concept and describing the kind of data that is expected. It's clearer for when a method has a parameter of type `Percentage` than of type `int`, and it's harder for it be invoked with an invalid value.

The same logic applies when declaring one user-defined type based on another user-defined type. When you have the same underlying data, but different sets of operations to perform, make two types.

### `iota` Is for Enumerations - Sometimes
Go doesn't have an enumeration type. Instead, it has `iota`, which lets you assign an increasing value to a set of constants.

When using `iota` the best practice is to first define a type based on `int` that will represent all of the valid values, then use a `const` block to define a set of values for  your type:
```go
type MailCategory int

const (
  Uncategorized MailCategory = iota // 0
  Personal // 1
  Spam // 2
  Social // 3
  Advertisements // 4
)
```
The first constant in the `const` block has the type specified and its value is set to `iota`. Every subsequent line has neither the type nor a value assigned to it. When the Go compiler sees this, it repeats the type and the assignment to all the subsequent constants in the block and increments the value of `iota` on each line.

The important thing to understand is that there is nothing in Go to stop you from creating additional values of your type. Also, if you insert a new identifier in the middle of your list of literals, all of the subsequent ones will be renumbered. This will break your application if those constants represent values in another system or database. Because of these limitations, `iota`-based enumerations only make sense when you care about being able to differentiate between a set of values and don't care about the value itself.

## Use Embedding for Composition
Go encourages code reuse via built-in support for composition and promotion:
```go
type Employee struct {
	Name string
	ID   string
}

func (e Employee) Description() string {
	return fmt.Sprintf("%s (%s)", e.Name, e.ID)
}

type Manager struct {
	Employee
	Reports []Employee
}

func (m Manager) FindNewEmployees() []Employee {
	// do business logic
}
```
Notice that `Manager` contains a field of type `Employee`, but no name is assigned to that field. This makes `Employee` an *embedded field*. Any fields of methods declared on an embedded field are *promoted* to the containing struct and can be invoked directly enabling the following: 
```go
func main() {
	m := Manager{
		Employee: Employee{
			Name: "Bob Bobson",
			ID:   "12345",
		},
		Reports: []Employee{},
	}
	fmt.Println(m.ID) // prints 12345
	fmt.Println(m.Description()) // prints Bob Bobson (12345)
}
```
If the containing struct has fields or methods with the same name as an embedded field, you need to use the embedded field's type to refer to the obscured fields or methods:
```go
type Inner struct {
	X int
}

type Outer struct {
	Inner
	X int
}

func main() {
	o := Outer{
		Inner: Inner{
			X: 10,
		},
		X: 20,
	}
	fmt.Println(o.X)       // 20
	fmt.Println(o.Inner.X) // 10
}
```
## Embedding is Not Inheritance


## A Quick Lesson on Interfaces

## Interfaces Are Type-Safe Duck Typing

## Embedding and Interfaces

## Accept Interfaces, Return Structs

## Interfaces and `nil`

## The Empty Interface Says Nothing

## Type Assertions and Type Switches

## Use Type Assertions and Type Switches Sparingly

## Function Types Are a Bridge to Interfaces

## Implicit Interfaces Make Dependency Injection Easier

## Wire

## Go Isn't Particularly Object-Oriented(and That's Great)

## Wrapping Up
