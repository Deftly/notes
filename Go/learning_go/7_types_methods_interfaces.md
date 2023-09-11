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
    - [`iota` Is for Enumerations... Sometimes](#iota-is-for-enumerations-sometimes)
  - [Use Embedding for Composition](#use-embedding-for-composition)
  - [Embedding Is Not Inheritance](#embedding-is-not-inheritance)
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

Go is a statically typed language with both built-in types and user-defined types. Like most modern languages, Go allows you to attach methods to types. It also has type abstraction, allowing you to write code that invokes methods without explicitly specifying the implementation.

However, Go's approach to methods, interfaces, and types is very different from most other languages. It encourages avoiding inheritance while encouraging composition. In this section we'll look at types, methods, and interfaces, and how to use them to build testable and maintainable programs.

## Types in Go
We previously saw how to define a struct type:

```go 
type Person struct {
    FirstName string
    LastName string 
    Age int
}
```

This should be read as declaring a user-defined type with the name `Person` to have the *underlying type* of the struct literal that follows. You can also use any primitive type or compound type literal to define a concrete type:

```go 
type Score int 
type Converter func(string)Score 
type TeamScores map[string]Score
```

You can declare a type at any block level from the package block down, but you can only access the type from within its scope. The exceptions are exported package block level types. We'll talk more about these in [section 9](./9_modules_packages_imports.md).

> An *abstract type* is one that specifies *what* a type should do, but now *how* it is done. A *concrete type* specifies what and how. This means that it has a specified way to store its data and provides an implementation of any methods declared on the type.

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

Method declarations look just like function declarations with the addition of the *receiver* specification. By convention, the receiver name is a short abbreviate of the type's name, usually its first letter. It is not idiomatic to use `this` or `self`.

Also like functions, method names cannot be overloaded. You can use the same method names for different types, but you can't use the same method name for different methods on the same type. Not reusing names is part of Go's philosophy of making clear what you code is doing.

We'll discuss packages in more detail in [section 9](./9_modules_packages_imports.md), but note that methods must be declared in the same package as their associated type. You can't add methods to types you don't control. Also, while you can define a method in a different file within the same package as the type declaration, it is best to keep your type definition and its associated methods together so it's easy to follow the implementation.

Method invocation is similar to that of other languages:

```go 
p := Person {
    FirstName: "Fred",
    LastName: "Fredson",
    Age: 52,
}
output := p.String()
```

### Pointer Receivers and Value Receivers
Methods in Go can have *pointer receiver*(the type is a pointer) or *value receivers*(the type is a value type). The following rules help us determine when to use each kind of receiver:
- If your method modifies the receiver, you *must* use a pointer receiver 
- If your method needs to handle `nil` instances, then it *must* be a pointer receiver
- If your method doesn't modify the receiver, you *can* use a value receiver

When a type has *any* pointer receiver methods, a common practice is to be consistent and use pointer receivers for *all* methods, even the ones that don't modify the receiver. Let's look at some examples:

```go 
type Counter struct {
	total       int
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
	fmt.Println(c.String()) // total: 1, last updated: 2023-08-30 09:44:32.311203667 -0700 PDT m=+0.000040501
}
```

One thing you might have noticed is that we were able to call the pointer receiver method even though `c` is a value type. When you use a pointer receiver with a local variable that's a value type, Go automatically converts it to a pointer type. In this case, `c.Increment()` is converted to `(&c).Increment()`.

However, be aware that the rules for passing values to functions still apply. If you pass a value type to a function and call a pointer receiver method on the passed value, you are invoking the method on a *copy*:

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

When we run this code we get the following output:

```shell 
in doUpdateWrong: total: 1, last updated: 2023-08-30 10:33:16.952380746 -0700 PDT m=+0.000017451
in main: total: 0, last updated: 0001-01-01 00:00:00 +0000 UTC
in doUpdateRight: total: 1, last updated: 2023-08-30 10:33:16.952463836 -0700 PDT m=+0.000100541
in main: total: 1, last updated: 2023-08-30 10:33:16.952463836 -0700 PDT m=+0.000100541
```

The parameter in `doUpdateRight` is of the type `*Counter`, which is a pointer instance. We can all both `Increment` and `String` on it because Go considers both pointer and value receiver methods to be in the *method set* for a pointer instance. For a value instance, only the value receiver methods are in the method set. This seems like a pedantic detail right now, but we'll come back to it when talking about interfaces in just a bit.

Do not write getter and setter methods for Go structs unless needed to meet an interface. Go encourages you to directly access a field. Reserve methods for business logic. The exceptions are when you need to update multiple fields as a single operation or when the update isn't a straightforward assignment of a new value.

### Code Your Methods for `nil` Instances
When you call a method on a `nil` instance Go will try to invoke the method. If it's a value receiver you'll get a panic since there is no value being pointed to by the pointer. If it's a pointer receiver it can work if the method is written to handle `nil` instances, an din some cases expecting a `nil` receiver makes the code simpler:

```go 
type IntTree struct {
	val         int
	left, right *IntTree
}

func (it *IntTree) Insert(val int) *IntTree {
	if it == nil {
		return &IntTree{val: val}
	}
	if val < it.val {
		it.left = it.left.Insert(val)
	} else if val > it.val {
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
```

> Note that while the `Contains` method does not modify the `*IntTree`, but it is declared with a pointer receiver. This is to support handling a `nil` receiver. A method with a value receiver can't check for `nil`.

The following code uses the tree:

```go 
func main() {
	var it *IntTree
	it = it.Insert(5)
	it = it.Insert(3)
	it = it.Insert(10)
	it = it.Insert(2)
	fmt.Println(it.Contains(2))  // true
	fmt.Println(it.Contains(12)) // false
}
```

Pointer receivers work just like pointer function parameters, it's a copy of the pointer that's passed into the method. Just like `nil` parameters passed to functions, if you change the copy of the pointer, you haven't changed the original. This means you can't write a pointer receiver method that handles `nil` and makes the original pointer no-nil. If your method has a pointer receiver and won't work for a `nil` receiver, check for `nil` and return an error(we'll discuss errors in [section 8](./8_errors.md))

### Methods Are Functions Too
Methods in Go are so much like functions that you can use a method as a replacement for a function any time there's a variable or parameter of a function type.

```go 
type Adder struct {
	start int
}

func (a Adder) AddTo(val int) int {
	return a.start + val
}
```

We create an instance of the type in the usual way and invoke its method:

```go 
myAdder := Adder{start: 10}
fmt.Println(myAdder.AddTo(5)) // prints 15
```

We can also assign the method to a variable or pass it to a parameter of type func(int)int.

```go 
f1 := myAdder.AddTo // This is called a method value
fmt.Println(f1(10)) //prints 20
```

A method value is a bit like a closure since it can access the value in the fields of the instance from which it was created.

You can also create a function from the type itself, this is called a *method expression*:

```go 
f2 := Adder.AddTo
fmt.Println(f2(myAdder, 15)) // prints 25
```

When using a method expression, the first parameter is the receiver for the method. 

### Functions Versus Methods
Any time your logic depends on values that are configured at startup or changed while your program is running, those values should be stored in a struct and that logic should be implemented as a method.

If your logic only depends on the input parameters, then it should be a function.

### Type Declarations Aren't Inheritance
In addition to declaring types based on built-in Go types and struct literals, you can also declare a user-defined type based on another user-defined type:

```go 
type HighScore Score 
type Employee Person
```

Declaring a type based on another type looks a bit like inheritance, but it isn't. The two types have the same underlying type but there is no hierarchy between these types. You can't assign an instance of type `HighScore` to variable type `Score` or vice versa without a type conversion, nor can you assign either of them to a variable of type `int` without a type conversion. Also, any methods defined on `Score` aren't defined on `HighScore`.

```go 
// assiging untyped constants is valid
var i int = 300 
var s Score = 100 
var hs HighScore = 200
hs = s              // compilation error
s = i               // compilation error
s = Score(i)        // valid
hs = HighScore(s)   // valid
```

For user-defined types whose underlying types are built-in types, a user-declared tyep can be used with operators for those types.

> A type conversion between types that share an underlying type keeps the same underlying storage but associates different methods.

### Types Are Executable Documentation
User defined types are documentation, they make code clearer by providing a name for a concept and describing the kind of data that is expected. It's clearer for someone reading your code when a method has a parameter of type `Percentage` than of type `int`, and it's harder for it to be invoked with an invalid value.

The same logic applies when declaring a user-defined type based on another user-defined type. When you have the same underlying data, but different sets of operations to perform, make two types.

### `iota` Is for Enumerations... Sometimes
Enumerations are a concept in many programming languages where you can specify that a type can only have a limited set of values. Go doesn't have enumerations, instead it has `iota`, which lets you assign an increasing value to a set of constants.

When using `iota`, the bet practice is to first define a type based on `int` that will represent all of the valid values:

```go 
type MailCategory int 
```

Next, use a `const` block to define a set of values for your type:

```go 
const (
    Uncategorized MailCategory = iota
    Personal 
    Spam 
    Social 
    Advertisements
)
```

The first constant in the `const` block has the type specified and its value is set to `iota`. Every subsequent line has neither the type nor a value assigned, the compiler will do this for you. In the above example the first constant(`Uncategorized`) is set to 0, the second constant(`Personal`) is set to 1 and so on. When a new constant block is created `iota` is set back to 0.

It's important to remember that there is nothing stopping you from creating additional values of your type. If you insert a new identifier in the middle of your list of literals, all subsequent ones will be renumbered. This will break the application in a subtle way if those constants represented values in another system or database. Because of these limitations it only makes sense to use `iota` when you care about being able to differentiate between a set of values and don't care what the value is behind the scenes. If the actual value matters, specify it explicitly.

## Use Embedding for Composition
The software engineering advice "Favor object composition over class inheritance" dates back to at least 1994. While Go doesn't have inheritance, it encourages code reuse via built-in support for composition and promotion:

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

The `Manager` struct contains a field of type `Employee`, but no name is assigned to it. This makes `Employee` an *embedded field*. Any fields or methods declared on the embedded field are *promoted* to the containing struct and can be invoked directly:

```go
m := Manager{
    Employee: Employee{
        Name: "Bob Bobson",
        ID:   "12345",
    },
    Reports: []Employee{},
}
fmt.Println(m.ID)            // prints 12345
fmt.Println(m.Description()) // prints Bob Bobson (12345)
```

> You can embed any type within a struct, not just another struct. This promotes the methods on the embedded type to the containing struct.

If the containing struct has fields or methods with the same name as the embedded field you need to use the embedded field's type to refer to the obscured fields or methods:

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
	fmt.Println(o.X)       // prints 20
	fmt.Println(o.Inner.X) // prints 10
}
```

## Embedding Is Not Inheritance
Built-in embedding support is rare in programming languages. Many developers try to understand embedding by treating it as inheritance, don't do this. You cannot assign a variable of type `Manager` to a variable of type `Employee`. If you want to access the `Employee` field in manager you must do so explicitly:

```go 
var eFail Employee := m        // compilation error: cannot use m (type Manager) as type Employee in assignment
var eOk Employee := m.Employee // valid
```

The methods on the embedded field don't know that they are embedded. If you have a method on an embedded field that calls another method on the embedded field, and the containing struct has a method of the same name, the method on the embedded field will not invoke the method on the contain struct:

```go 
type Inner struct {
	A int
}

func (i Inner) IntPrinter(val int) string {
	return fmt.Sprintf("Inner: %d", val)
}

func (i Inner) Double() string {
	return i.IntPrinter(i.A * 2)
}

type Outer struct {
	Inner
	S string
}

func (o Outer) IntPrinter(val int) string {
	return fmt.Sprintf("Outer: %d", val)
}

func main() {
	o := Outer{
		Inner: Inner{
			A: 10,
		},
		S: "Hello",
	}
	fmt.Println(o.Double()) // Inner: 20
}
```

## A Quick Lesson on Interfaces
We'll start with how to declare an interface. Like other user-defined types, you use the `type` keyword. Here's the definition of the `Stringer` interface in the `fmt` package:

```go 
type String interface {
    String() string
}
```

In an interface declaration, an interface literal appears after the name of the interface type. It lists the methods that must be implemented by a concrete type to meet the interface. The methods defined by the interface are called the method set of the interface.

Like other types interfaces can be declared in any block and usually have names ending in "er". Examples: `io.Reader`, `io.Closer`, `json.Marshaler`, `http.Handler`.

## Interfaces Are Type-Safe Duck Typing
What makes Go's interfaces special is that they implemented *implicitly*. A concrete type does not declare that it implements an interface. If the method set for a concrete type contains all of the methods in the method set for an interface, the concrete type implements that interface. This means that the concrete type can be assigned to a variable or field declared to be of the type of the interface.

This implicit behavior enables both type-safety and decoupling, bridging the functionality in both static and dynamic languages. 

The reason languages have interfaces is to allow you to "Program to an interface, not an implementation". Doing so allows you to depend on behavior, not an implementation, meaning you can swap implementations as needed.

Dynamically typed languages like Python, Ruby, and JavaScript don't have interfaces. Instead, they have "duck typing", which is based on the expression "If it walks like a duck and quacks like a duck, it's a duck". The concept is that you can pass an instance of a type as a parameter to a function as long as the function can find a method to invoke that it expects:

```python 
class Logic:
def process(self, data):
    # business logic

def program(logic):
    # get data from somewhere
    logic.process(data)

logicToUse = Logic()
program(logicToUse)
```

The problem here is that it can be hard to know exactly what functionality should be expected, you have to trace through the code to figure out what the actual dependencies are. Java developers use a different pattern. They define an interface, create an implementation of the interface, but only refer to the interface in the client code:

```java 
public interface Logic {
    String process(String data);
}

public class LogicImpl implements Logic {
    public String process(String data) {
        // business logic
    }
}

public class Client {
    private final Logic logic; // this type is the interface, not the implementation
    
    public Client(Logic logic) {
        this.logic = logic;
    }
    
    public void program() {
        // get data from somewhere
        this.logic(data);
    }
}

public static void main(String[] args) {
    Logic logic = new LogicImpl();
    Client client = new Client(logic);
    client.program();
}
```

Dynamic language developers look at the explicit interfaces in Java and don't see how you can easily refactor your code over time when you have explicit dependencies. Switching to a new implementation from a different provider means rewriting your code to depend on a new interface.

Go is a blend of the previous two styles:

```go 
type LogicProvider struct {}

func (lp LogicProvider) Process(data string) string {
    // business logic
}

type Logic interface {
    Process(data string) string 
}

type Client struct {
    L Logic
}

func(c Client) Program() {
    // get data from somewhere
    c.L.Process(data)
}

main() {
    c := Client{
        L: LogicProvider{},
    }
    c.Program()
}
```

In the Go code, there is an interface, but only the caller(`Client`) knows about it. There is nothing declared on `LogicProvider` to indicate that it meets the interface. This is sufficient to both allow a new logic provider in the future and provide executable documentation to ensure that any type passed into the client will match the client's need.

Having a standard interface is powerful. If you write code to work with `io.Reader` and `io.Writer` it will function correctly whether it is writing to a file on local disk or a value in memory. 

Using standard interfaces also encourages the *decorator pattern*. It is common in Go to write factory functions that take in an instance of an interface and return another type that implements the same interface. For example, say you have a function with the following definition:

```go 
func process(r io.Reader) error
```

You can process data from a file with the following code:

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

Now the same code that was reading from an uncompressed file is reading from a compressed file instead.

It's perfectly fine for a type that meets an interface to specify additional methods that aren't part of the interface. One set of client code may not care about those methods, but others do. For example, the `io.File` type also meets the `io.Writer` interface. If your code only cares about reading from a file, use the `io.Reader` interface to refer to the file instance and ignore the other methods.

## Embedding and Interfaces
Just like you can embed a type in a struct, you can also embed an interface in an interface:

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
You'll often hear experienced Go developers say that your code should "Accept interfaces, return structs". We've covered why functions should accept interfaces: they make your code more flexible and explicitly declare exactly what functionality is being used.

If you create an API that returns interfaces, you are losing one of the main advantages of implicit interfaces: decoupling. You want to limit the third-party interfaces that you client code depends on because your code is now permanently dependent on the module that contains those interfaces, as well as any dependencies of that module(more on modules in [section 9](./9_modules_packages_imports.md)), and so on.

While depending on concrete instances can lead to dependencies, using a dependency injection layer(We'll talk more about dependency injection later in this [section](./7_types_methods_interfaces.md#implicit-interfaces-make-dependency-injection-easier)) can limit the effect.

There is one potential drawback to this pattern. Returning a struct avoid a heap allocation. However, when invoking a function with parameters of interface types, a heap allocation occurs for each of the interface parameters. Figuring out the trade-off between better abstraction and better performance should be done over the life of your program. Write your code so that it is readable and maintainable. If you find that your program is too slow *and* you have profiled it *and* you have determined that the performance problems are due to heap allocation caused by an interface parameter, then you should rewrite the function to use a concrete type parameter.

## Interfaces and `nil`
In order for an interface to be considered `nil` *both* the type and the value must be `nil`:

```go 
var s *string 
fmt.Println(s == nil) // prints true
var i interface{} 
fmt.Println(i == nil) // prints true
i = s 
fmt.Println(i == nil) // prints false
```

In the Go runtime, interfaces are implemented as a pair of pointers, one to the underlying type and one to the underlying value. As long as the type is non-nil, the interface is non-nil.

What `nil` indicates for an interface is whether or not you can invoke methods on it. If an interface is `nil`, invoking any methods on it triggers a panic. If an interface is non-nil, you can invoke methods on it, however, if the value is `nil` and the methods of the assigned type don't handle `nil` you could still trigger a panic.

Since an interface instance with a non-nil type is not equal to `nil`, it is not straightforward to tell whether or not the value associated with the interface is `nil` when the type is non-nil. You must use reflection(discussed in [section 14](./14_here_there_be_dragons:reflect_unsafe_and_cgo.md#use-reflection-to-check-if-an-interfaces-value-is-nil)) to find out.

## The Empty Interface Says Nothing
Sometimes you need a way to say that a variable could store a value of any type, Go uses `interface{}` to represent this:

```go 
var i interface{}
i = 20 
i = "hello"
i = struct {
    FirstName string 
    LastName string
}{"Fred", "Fredson"}
```

Any empty interface type simply states that the variable can store any value whose type implements zero or more methods, which happens to match every type in Go. A common use of the empty interface is as a placeholder for data of uncertain schema that's read from an external source, like a JSON file: 

```go 
data := map[string]interface{}{}
contents, err := ioutil.ReadFile("testdata/sample.json")
if err != nil {
    return err
}
defer contents.Close()
json.Unmarshal(contents, &data) // the contents are now in the data map
```

If you see a function that takes in an empty interface, it's likely that it is using reflection([section 14](./14_here_there_be_dragons:reflect_unsafe_and_cgo.md)) to either populate or read the value. These situations should be relatively rare. Avoid using `interface{}`. Go is designed as a strongly typed language and attempts to work around this are unidiomatic.

If you find yourself having to store a value in an empty interface, you might be wondering how to read the value back again. To do this, we need to look at type assertions and type switches.

## Type Assertions and Type Switches


## Use Type Assertions and Type Switches Sparingly

## Function Types Are a Bridge to Interfaces

## Implicit Interfaces Make Dependency Injection Easier

## Wire

## Go Isn't Particularly Object-Oriented(and That's Great)

## Wrapping Up
