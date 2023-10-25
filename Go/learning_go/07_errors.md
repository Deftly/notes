# Errors

<!--toc:start-->
- [Errors](#errors)
  - [How to Handle Errors: The Basics](#how-to-handle-errors-the-basics)
  - [Use Strings for Simple Errors](#use-strings-for-simple-errors)
  - [Sentinel Errors](#sentinel-errors)
  - [Errors Are Values](#errors-are-values)
  - [Wrapping Errors](#wrapping-errors)
  - [Using `Is` and `As`](#using-is-and-as)
  - [Wrapping Errors with `defer`](#wrapping-errors-with-defer)
  - [Using `panic` and `recover`](#using-panic-and-recover)
  - [Getting a Stack Trace from an Error](#getting-a-stack-trace-from-an-error)
  - [Wrapping Up](#wrapping-up)
<!--toc:end-->

## How to Handle Errors: The Basics
Go handles errors by returning a value of type `error` as the last return value for a function(this is a conventions that should never be breached). If a function executes as expected, `nil` is returned for the error parameter. If something goes wrong, an error value is returned instead:
```go
func calcRemainderAndMod(numerator, denominator int) (int, int, error) {
  if denominator == 0 {
    return 0, 0, errors.New("denominator is 0")
  }
  return numerator / denominator, numerator % denominator, nil
}
```
Unlike other languages, Go doesn't have special constructs to detect if an error was returned. Whenever a function returns, use an `if` statement to check the error value:
```go
func main() {
  numerator := 20
  denominator := 3
  remainder, mod, err := calcRemainderAndMod(numerator, denominator)
  if err != nil {
    fmt.Println(err)
    os.Exit(1)
  }
  fmt.Println(remainder, mod)
}
```
Error is a built-in interface that defines a single method:
```go
type error interface {
  Error() string
}
```
There are two main reasons why Go uses a returned error instead of thrown exceptions. First exceptions add at least one new code path through the code. These paths are sometimes unclear, especially in languages whose functions don't include a declaration that an exception is possible. 

The second reason is more subtle. Remember that the Go compiler requires that all variables must be read. And since errors are returned values it forces developers to either check and handle error conditions or make it explicit that they are ignoring errors by using an underscore(_) for the returned error value.

## Use Strings for Simple Errors
Go's standard library provides two ways to create an error from a string. The first is the `error.New` function. It takes in a `string` and returns an `error`. This string is returned when you call the `Error` method on the returned error instance. If you pass an error to `fmt.Println` it calls the `Error` method automatically:
```go
func doubleEven(i int) (int, error) {
  if i % 2 != 0 {
    return 0, errors.New("only even numbers are processed")
  }
  return i * 2, nil
}
```
The second way is to use the `fmt.Errorf` function. This allows you to use all the formatting verbs for `fmt.Printf` to create an error.

## Sentinel Errors
Some errors are meant to signal the processing cannot continue due to a problem with the current state, these are known as sentinel errors.

Sentinel errors are one of the few variables that are declared at the package level. By convention their names start with `Err`(with the notable exception of `io.EOF`). They should be treated as read-only, this isn't enforced by the compiler but it is a programming error to change their value.

The standard library include a package for processing ZIP files, `archive/zip`. This package defines several sentinel errors, including `ErrFormat`, which is returned when data that doesn't represent a ZIP file is passed in:
```go
func main() {
  data := []byte("This is not a zip file")
  notAZipFile := bytes.NewReader(data)
  _, err := zip.NewReader(notAZipFile, int64(len(data)))
  if err == zip.ErrFormat {
    fmt.Println("Told you so")
  }
}
```
Be sure you need a sentinel error before you define one. Once you define one, it is part of your public API and you have committed to it being available for all future backward-compatible releases. It's far better to reuse one of the existing ones in the standard library or to define an error type that includes information about the condition that caused the error to be returned.

If you have an error condition that indicates a specific state has been reached in your application where no further processing is possible and no contextual information needs to be used to explain the error state, a sentinel error is the correct choice.

## Errors Are Values
Since `error` is an interface, you can define your own errors that include additional information for logging or error handling. For example, you might want to include a status code as part of the error to indicate the kind of error that should be reported back to the user:
```go
type Status int

const (
	InvalidLogin Status = iota + 1
	NotFound
)
```
Next, define a `StatusErr` to hold this value:
```go
type StatusErr struct {
	Status  Status
	Message string
}

func (se StatusErr) Error() string {
	return se.Message
}
```
Now we can use `StatusErr` to provide more details about what went wrong:
```go
func LoginAndGetData(uid, pwd, file string) ([]byte, error) {
	err := login(uid, pwd)
	if err != nil {
		return nil, StatusErr{
			Status:  InvalidLogin,
			Message: fmt.Sprintf("invalid credentials for user %s", uid),
		}
	}
	data, err := getData(file)
	if err != nil {
		return nil, StatusErr{
			Status:  NotFound,
			Message: fmt.Sprintf("file %s not found", file),
		}
	}
	return data, nil
}
```
Even when you define your own custom error types, always use `error` as the return type for the error result. This allows you to return different types of errors from your function and allows callers of your function to choose not to depend on the specific error type.

## Wrapping Errors

## Using `Is` and `As`

## Wrapping Errors with `defer`

## Using `panic` and `recover`

## Getting a Stack Trace from an Error

## Wrapping Up
