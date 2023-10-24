# Types, Methods, and Interfaces

<!--toc:start-->
- [Types, Methods, and Interfaces](#types-methods-and-interfaces)
  - [Types in Go](#types-in-go)
  - [Methods](#methods)
    - [Pointer Receivers and Value Receivers](#pointer-receivers-and-value-receivers)
    - [Code Your Methods for `nil` Instances](#code-your-methods-for-nil-instances)
    - [Methods Are Functions Too](#methods-are-functions-too)
    - [Functions Versus Methods](#functions-versus-methods)
    - [Type Declarations Aren't Inheritance](#type-declarations-arent-inheritance)
    - [Types Are Executable Documentation](#types-are-executable-documentation)
    - [`iota` Is for Enumerations - Sometimes](#iota-is-for-enumerations-sometimes)
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
  - [Implicit Interfaces Make Dependency Injection Easier](#implicit-interfaces-make-dependency-injection-easier)
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
Built-in embedding supports is rate in programming languages and many developer try to understand embedding by treating it as inheritance, don't do this. You cannot assign a variable of type `Manager` to a variable of type `Employee`. If you want to access the `Employee` field in `Manager`, you must do so explicitly:
```go
var eFail Employee = m // error: cannot use m (type Manager) as type Employee in assignment
var eOk Employee = m.Employee
```
While embedding one concrete type inside another won't allow you to treat the outer type as the inner type, the methods on an embedded field do count toward the *method set* of the containing struct. This means they can allow the containing struct implement an interface.

## A Quick Lesson on Interfaces
Implicit interfaces are the only abstract type in Go. You declare an interface like other user-defined types using the `type` keyword:
```go
typer Stringer interface {
  String() string
}
```
In an interface declaration, an interface literal appears after the name of the interface which lists the methods that must be implemented by a concrete type to meet the interface. The methods defined by an interface are called the method set of the interface.

Interfaces are usually named with "er" endings: `fmt.Stringer`, `io.Reader`, `io.Closter`, `io.ReadCloser`, `json.Marshaler`, `http.Handler`.

## Interfaces Are Type-Safe Duck Typing
What makes Go's interfaces special is that they are implemented *implicitly*. A concrete type does not declare that it implements an interface. If a method set for a concrete type contains all the methods in the method set of for an interface, the concrete type implements the interface. This means that the concrete type can be assigned to a variable or field declared to be of the type of the interface.

This implicit behavior of types in Go enables both type-safely and decoupling, bridging the functionality in both static and dynamic languages.

The reason why languages have interfaces is to allow you to "Program to an interface, not an implementation". This lets you depend on a behavior, not an implementation, meaning you can swap implementations as needed.

Dynamically typed languages like Python or JavaScript don't have interfaces. Instead, those developers use "duck typing". This is based on the expression "If it walks like a duck and quacks like a duck, it's a duck". The concept is that you can pass an instance of a type as parameter to a function as longs as the function can find a method to invoke that it expects:
```python
def animal_sound(animal):
  animal.speak()

class Dog:
  def speak(self):
    print("Woof")

class Cat:
  def speak(self):
    print("Meow")

d = Dog()
c = Cat()
animal_sound(d) # Output: Woof
animal_sound(c) # Output: Meow
```
Java developers use a different pattern:
```java 
// Explicit Interfaces in Java
interface Animal {
    void speak();
}

class Dog implements Animal {
    public void speak() {
        System.out.println("Woof");
    }
}

class Cat implements Animal {
    public void speak() {
        System.out.println("Meow");
    }
}

public class Main {
    public static void animalSound(Animal animal) {
        animal.speak();
    }

    public static void main(String[] args) {
        Animal d = new Dog();
        Animal c = new Cat();

        animalSound(d);  // Output: Woof
        animalSound(c);  // Output: Meow
    }
}
```
Go is a blend of the previous two styles:
```go
// Animal interface
type Animal interface {
    Speak() string
}

type Dog struct {}

func (d Dog) Speak() string {
    return "Woof"
}

type Cat struct {}

func (c Cat) Speak() string {
    return "Meow"
}

// animalSound function
func animalSound(a Animal) {
    fmt.Println(a.Speak())
}

func main() {
    var d Animal = Dog{}
    var c Animal = Cat{}

    animalSound(d)  // Output: Woof
    animalSound(c)  // Output: Meow
}
```
We've seen several interfaces in the standard library that are used for input and output. Having a standard interface is powerful. If you write your code to work with `io.Reader` and `io.Writer`, it will function correctly whether it is writing to a file on local disk disk or a value in memory.

Interfaces encourage the *decorator pattern*. It is common in Go to write factory functions that take in an instance of an interface and return another type that implements the same interface. For example, say you have the following function:
```go
func process(r io.Reader) error
```
You can process data from a file with the following:
```go
r, err := os.Open(fileName)
if err != nil {
  return err
}
defer r.Close()
return process(r)
```
The `os.File` instance returned by `os.Open` meets the `io.Reader` interface and can be used in any code that reads in data. If the file is gzip-compressed, you can wrap the `io.Reader` in another `io.Reader`:
```go
r, err := os.Open(fileName)
if err != nil {
  return err
}
defer r.Close()
gz, err := gzip.NewReader(r)
if err != nil {
  return err
}
defer gz.Close()
return process(gz)
```
> **_NOTE:_** If there's an interface in the standard library that describes what your code needs, use it!

It's fine for a type that meets an interface to specify additional methods that aren't part of the interface. For example, the `io.File` type also meets the `io.Writer` interface. If your code only cares about reading from a file, use the `io.Reader` interface to refer to the file instance and ignore the other methods.

## Embedding and Interfaces
Just like you can embed a type in a struct, you can embed an interface in an interface. For example, `io.ReadCloser` interface is built on an `io.Reader` and an `io.Closer`:
```go
type Reader interface {
  Read(p []byte) (n int, err error)
}
type Closer interface {
  Close() error
}
type ReadCloser interface {
  Reader
  Closer
}
```
## Accept Interfaces, Return Structs
The saying "Accept interfaces, return structs" means that business logic invoked by your functions should be invoked via interfaces, but the output of your functions should be a concrete type. Functions that accept interfaces make your code more flexible and explicitly declare exactly what functionality is being used.

If you return a concrete type new methods and fields can be added without breaking existing code. The same is not true for an interface. Adding a new method to an interface means that you need to update all existing implementations of the interface or your code breaks.

Instead of writing a single factory function that returns different instances behind an interface based on input parameters, try to write separate factory functions for each concrete type. In some situations(such as a parser that can return one or more different kinds of tokens), it's unavoidable and you have no choice but to return an interface.

Errors are the exception to this rule. Go functions and methods declare a return parameter of the `error` interface type. In the case of `error`, it's quite likely that different implementations of the interface could be returned so you need an interface to handle all possible options, as interfaces are the only abstract type in Go.

## Interfaces and `nil`
In order for an interface to be considered `nil` *both* the type and the value must be `nil`:
```go
var s *string 
fmt.Println(s == nil) // true
var i interface{}
fmt.Println(i == nil) // true
i = s
fmt.Println(i == nil) // false
```
In the Go runtime, interfaces are implemented as a pair of pointers, one to the underlying type and one to the underlying value. As long as the type is non-nil, the interface is non-nil(Since you can't have a variable without a type, if the value pointer is non-nil, the type pointer is always non-nil).

What `nil` indicates for an interface is whether or not you can invoke methods on it. We saw earlier that you can invoke methods on `nil` concrete instances, so it makes sense that you can invoke methods on an interface that was assigned a `nil` concrete instance. If an interface is `nil`, invoking any methods on it triggers a panic. If an interface is non-nil, you can invoke methods on it(note that if the value is `nil` and the methods of the assigned type don't handle `nil`, you could still trigger a panic).

## The Empty Interface Says Nothing
There are instance where you need a way to say that a variable could store any value, for this Go uses the empty interface:
```go
var i interface{}
i = 20
i = "hello"
i = struct {
  FirstName string
  LastName string
}{"Fred", "Fredson"}
```
`interface{}` isn't a special case syntax. It simply states that the variable can store any value whose type implements zero or more methods, which happens to match every type in Go. Because an empty interface doesn't tell you anything about the value it represents there isn't a lot you can do with it. A common use of the empty interface is a placeholder for data of uncertain schema that's read from an external source:
```go
data := map[string]interface{}{}
contents, err := ioutil.ReadFile("testdata/sample.json")
if err != nil {
  return err
}
defer contents.Close()
json.Unmarshal(contents, &data)
```
If you see a function that takes in an empty interface, it's likely that it is using reflection(we'll cover this in a later section) to either populate or read the value.

Avoid using `interface{}` when possible, Go is designed as a strongly typed language and attempts to work around this are unidiomatic.

## Type Assertions and Type Switches
Go provides two ways to see if a variable of an interface type has a specific concrete type or if the concrete type implements another interface. We'll start with *type assertions*. A type assertion names the concrete type that implemented the interface, or names another interface that is also implemented by the concrete type underlying an interface:
```go
type MyInt int

func main() {
	var i interface{}
	var mine MyInt = 20
	i = mine
	i2 := i.(MyInt)
	fmt.Println(i2 + 4)
}
```
If the type conversion is wrong your code will panic:
```go
i2 := i.(string) // panic: interface conversion: interface {} is main.MyInt, not string
fmt.Println(i2)
```
Even if two types share an underlying type, a type conversion must match the type of the underlying value:
```go
i2 := i.(int) // panic: interface conversion: interface {} is main.MyInt, not int 
fmt.Println(i2 + 1)
```
To prevent crashing we can use the comma ok idiom like we saw in previous sections:
```go
i2, ok := i.(int) // ok set to true if conversion was successful
if !ok {
  return fmt.Errorf("unexpected type for %v", i)
}
fmt.Println(i2 + 1)
```
> **_NOTE:_** Type conversions can be applied to both concrete types and interfaces and are checked at compile time. Type assertions can only be applied to interface types and are checked at runtime. Conversions change, assertions reveal.

Even if you are absolutely certain that your type assertion is valid use the comma ok idiom. You don't know how other people(or you in 6 months) will reuse your code.

When an interface could be one of multiple possible types, use a *type switch* instead:
```go
func doThings(i interface{}) {
	switch j := i.(type) {
	case nil:
		// i is nil, type of j is interface{}
	case int:
		// j is of type int
	case MyInt:
		// j is of type MyInt
	case io.Reader:
		// j is of type io.Reader
	case string:
		// j is a string
	case bool, rune:
		// i is either a bool or rune, so j is of type interface
	default:
		// no idea what i is, so j is of type interface{}
	}
}
```
> **_NOTE:_** Since the purpose of a type `switch` is to derive a new variable from an existing one, it is idiomatic to assign the variable being switched on to a variable of the same name(`i := i.(type)`), making this one of the few places where shadowing is a good idea.

## Use Type Assertions and Type Switches Sparingly
For the most part, treat a parameter or return value as the type that was supplied and not what else it could be. Otherwise, your function's API isn't accurately describing what types it needs to perform its task. If you need a different type then it should be specified.

One common use of type assertion is to see if the concrete type behind the interface also implements another interface. For example, the standard library uses this technique to allow more efficient copies when the `io.Copy` function is called. This function has two parameters of types `io.Writer` and `io.Reader` and calls the `io.copyBuffer` function to do its work. If the `io.Writer` parameter also implements `io.WriterTo`, or the `io.Reader` parameter implements `io.ReaderFrom`, most of the work of the function can be skipped:
```go
// copyBuffer is the actual implementation of Copy and CopyBuffer.
// if buf is nil, one is allocated.
func copyBuffer(dst Writer, src Reader, buf []byte) (written int64, err error) {
	// If the reader has a WriteTo method, use it to do the copy.
	// Avoids an allocation and a copy.
	if wt, ok := src.(WriterTo); ok {
		return wt.WriteTo(dst)
	}
	// Similarly, if the writer has a ReadFrom method, use it to do the copy.
	if rt, ok := dst.(ReaderFrom); ok {
		return rt.ReadFrom(src)
	}
  // function continues ...
}
```
Type switch statements provide the ability to differentiate between multiple implementations of an interface that require different processing:
```go
func walkTree(t *treeNode) (int, error) {
	switch val := t.val.(type) {
	case nil:
		return 0, errors.New("invalid expression")
	case number:
		// we know that t.val is of type number, so return the
		// int value
		return int(val), nil
	case operator:
		// We know that t.val is of type operator, so
		// find the values of the left and right children, then
		// call the process() method on operator to return the
		left, err := walkTree(t.lchild)
		if err != nil {
			return 0, err
		}
		right, err := walkTree(t.rchild)
		if err != nil {
			return 0, err
		}
		return val.process(left, right), nil
	default:
		// If a new treeVal type is defined, but walkTree wasn't updated
		// to process it, this detects it
		return 0, errors.New("unknown node type")
	}
}
```
## Implicit Interfaces Make Dependency Injection Easier
Dependency injection is the concept that you code should explicitly specify the functionality it needs to perform its task. One of the surprising benefits of Go's implicit interfaces is that they make dependency injection an excellent way to decouple your code. Developers in other languages often use large, complicated frameworks to implement dependency injection. In Go it's easy to implement dependency injection without any additional libraries.

Go's approach to interfaces is "implicit", meaning that a type doesn't need to explicitly declare that it implements an interface, it only needs to have the methods that the interface requires. This has several advantages when it comes to dependency injection:
- **Simplicity**: You define an interface where you need it, specifying only the methods you'll actually use. This makes the contract between dependencies explicit and easy to understand.
- **Easy Refactoring**: If you start with a concrete type and later decide you need an abstraction, you can define an interface that matches the existing method signatures. This means you can make your code more abstract without breaking existing functionality.
- **Localized Interfaces**: In Go, it's idiomatic to define interfaces where they are used, rather than where a type is implemented. This localization of interfaces to the client code makes it easier to manage dependencies and understand their context.
- **Interface Composition**: Go allows you to compose smaller interfaces to create more complex ones, making it easier to inject exactly the functionality a component needs:
```go
type ReadWriteCloser interface {
  Reader
  Writer
  Closer
}
```
Now let's look at a simple example:
```go
type Writer interface {
	Write([]byte) (int, error)
}

func SaveFile(w Writer, content string) {
	w.Write([]byte(content))
}

type FileWriter struct{}

func (FileWriter) Write(bytes []byte) (int, error) {
	fmt.Println("Writing to a file:", string(bytes))
	return len(bytes), nil
}

type InMemoryWriter struct{}

func (InMemoryWriter) Write(bytes []byte) (int, error) {
	fmt.Println("Writing to memory:", string(bytes))
	return len(bytes), nil
}

func main() {
	fw := FileWriter{}
	SaveFile(fw, "File content")

	mw := InMemoryWriter{}
	SaveFile(mw, "Memory content")
}
```
## Go Isn't Particularly Object-Oriented(and That's Great)
It's hard to categorize Go as a particular style of language. It clearly isn't a strictly procedural language. At the same time, Go's lack of method overriding, inheritance, and objects means that it is not a particularly object-oriented language either. Go has function types and closures, but it isn't a functional language. Go borrows concepts from many places with the overriding goal of creating a language that is simple, readable, and maintainable.

## Wrapping Up
This section covered, types, methods, interfaces, and their best practices. The [next section](./07_errors.md) will cover how to use one of Go's most controversial features: errors.
