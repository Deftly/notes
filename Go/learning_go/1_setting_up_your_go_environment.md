# Setting Up Your Go Environment

## Installing the Go Tools
To write Go code you need to download and install the Go development tools which can be found on the [Go website](https://go.dev/dl/). The *.pkg* installer for Mac and *.msi* installer for Windows automatically installs Go in the correct location, remove any old installations, and put the Go binary in the default executable path. The Linux installers are gzipped tar files and expand to a directory named *go*. Copy this directory to */usr/local* and add */usr/local/go/bin* to your *$PATH* so that the `go` command is accessible:

```shell
$ tar -C /usr/local -xzf go1.20.4.linux-amd64.tar.gz
$ echo "export PATH=$PATH:/usr/local/go/bin" >> $HOME/.bashrc
$ source $HOME/.bashrc
```

> Go programs compile to a single binary and do not require any additional software to be installed in order to run them. The Go development tools are only needed on machines that build Go programs.

You can validate that your environment is set up correctly using the following:

```shell
$ go version
go version go1.20.4 linux/amd64
```

## The Go Workspace
For modern Go development you are free to organize your projects as you see fit. However, Go still expects there to be a single workspace for third-party Go tools installed via `go install`. By default this workspace is located in *$HOME/go*, with source code for these tools stored in *$HOME/go/src* and the compiled binaries in *$HOME/go/bin*. This can be changed by setting the *$GOPATH* environment variable.

Explicitly defining the *$GOPATH* and putting *$GOPATH/bin* in your executable path are considered good practices and will make it easier to run third-party tools installed via `go install`. On a Unix-like system using `bash` this can done by adding the following lines to your *.bashrc*:

```bash
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
```

On Windows(☹️) this can done with the following commands:

```shell
setx GOPATH %USERPROFILE%\go
setx path "%path%;%USERPROFILE%\bin"
```

There are other environment variables that are recognized by the `go` tool, you can get a complete list along with a brief description of each variable using the `go env` command.

## The `go` Command
Go comes with many development tools which can be accessed via the `go` command. These include a compiler, code formatter, linter, dependency manager, test runner, and more.

### `go run` 
We'll start with `go run`. Create a directory with a file called *hello.go*:

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, world!")
}
```

After the file is saved we can run:

```shell
$ go run hello.go
Hello, world!
```

If you look inside the directory you'll see only the *hello.go* file and no binary, Go is a compiled language so what gives? The `go run` command does in compile your code into a binary, however, the binary is built in a temporary directory. After it is built, the binary is executed and then deleted once it is finished. This makes the `go run` command useful for testing out small programs or using Go like a scripting language.

### `go build`
Most of the time you want to build a binary for later use, this is where you use the `go build` command.

```shell
go build hello.go
```

This creates an executable called `hello`(or *hello.exe* on Windows ☹️) in the current directory. The name of the binary matches the name of the file or package that you passed in. If you want a different name for your application, or stare it in a different location use the `-o` flag like so:

```shell
go build -o hello_world hello.go
```

## Getting Third-Party Go Tools
While some people distribute their Go programs as pre-compiled binaries, tools written in Go can also be build from source and installed into your Go workspace via the `go install` command.

Go's method for publishing code is different than most other languages, it doesn't rely on a centrally hosted service like Maven Central for Java or the NPM registry for JavaScript. Instead code is shared via source code repositories. The `go install` command takes an argument, which is the location of the source code repository of the project you want to install followed by an @ and the version of the tool you want(for the latest version use @latest). It then downloads, compiles, and installs the tool into your *$GOPATH/bin* directory. 

Let's look at an example. There is a great Go tool called `hey` that load tests HTTP servers. Here's how we install `hey` using `go install`:

```shell
$ go install github.com/rakyll/hey@latest
go: downloading github.com/rakyll/hey v0.1.4
go: downloading golang.org/x/net v0.0.0-20181017193950-04a2e542c03f
go: downloading golang.org/x/text v0.3.0
```

This downloads `hey` and all its dependencies, builds the program, and installed the binary in your *$GOPATH/bin* directory.

Now we can run it with:

```shell
hey https://www.google.com
Summary:
    Total:        0.6864 secs
    Slowest:      0.3148 secs
    Fastest:      0.0696 secs
    Average:      0.1198 secs
    Requests/sec: 291.3862
```

If you have already installed a tool and want to update to a newer version, rerun `go install` with the newer version specified or with @latest.

## Makefiles
Modern software development relies on repeatable, automatable builds that can be run by anyone, anywhere, at any time. The way to do this is to use some kind of script to specify your build steps. Go developers use `make` for this.

Here's a sample Makefile to add to our very simple project:

```makefile
.DEFAULT_GOAL := build

fmt:
        go fmt ./...
.PHONY:fmt

lint: fmt
        golint ./...
.PHONY:lint

vet: fmt
        go vet ./...
.PHONY:vet

build: vet
        go build hello.go
.PHONY:build
```

Each possible operation is called a *target* and the `.DEFAULT_GOAL` defines which target is run when no target is specified. In the above case, we are going to run the `build` target. The word before each colon is the name of the target and any words that come after the colon are the other targets must be run before the specified target runs. The tasks that are performed by the target are on the indented lines after the target. The `.PHONY` line keeps `make` from getting confused if you ever create a directory in your project with the same name as a target.

We can now use the makefile like so:

```shell
$ make
go fmt ./...
go vet ./...
go build hello.go
```

With this single command we make sure the code was formatted correctly, vet the code for non-obvious errors, and compile.

One drawback of Makefiles is that they are not supported out-of-the-box on Windows(☹️). You can install make using a package manager like [Chocolatey](https://chocolatey.org/). The command is `choco install make`.

## Staying Up to Date
As with all programming languages, there are periodic updates to the Go development tools. Go programs are native binaries that don't rely on a separate runtime so updating your development won't cause any problems for existing programs. 

Go has a new major release roughly every 6 months with there being minor releases with bug and security fixes released as needed. Go releases tend to be incremental and the GO team is committed to maintaining backwards compatibility. The [Go Compatibility Promise](https://go.dev/doc/go1compat) is a detailed description of how the Go team plans to avoid breaking Go code.

## Wrapping Up
So far we covered how to install and configure a Go development environment as well as some of the tools for building Go programs. In the [next section](./2_primitive_types_and_declarations.md) we'll cover the built-in types in Go and how to declare variables.

