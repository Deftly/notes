# Getting Started

## Installation
The following command downloads a script and starts the installation of the `rustup` tool, which installs the latest stable version of Rust.

```bash
curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

You will also need a *linker*, which is a program that Rust uses to join its compiled outputs into one file. You will likely already have one like GCC or Clang.

### Updating and Uninstalling
Once Rust is installed via `rustup`, updating to a newly release version is easy.

```bash
rustup update
```

To uninstall Rust and `rustup`, run the following:

```bash
rustup self uninstall
```

### Local Documentation
The installation of Rust also includes a local copy of the documentation so that you can read it offline. Run `rustup doc` to open the local documentation in your browser.

## Hello, World!
We'll start with a simple hello world example, create a new source file and call it *main.rs*. Rust files always end with the *.rs* extension. It is also convention to use an underscore to separate words in your file name.

```Rust
fn main() {
  println!("Hello, world!");
}
```

To run the file simply enter the following commands:

```bash
rustc main.rs
./main
Hello, world!
```

### Anatomy of a Rust Program
```Rust
fn main() {

}
```

These lines define a function named `main`. The `main` function is special: it is always the first code that runs in every executable Rust program. The first line declares a function named `main` that has no parameters and returns nothing. 

The function body is wrapped in `{}`. Rust requires curly brackets around all function bodies.

> **_NOTE:_** If you want to stick to a standard style across Rust projects, you can use an automatic formatter tool called `rustfmt`.

The body of the `main` function holds the following code:

```Rust
    println!("Hello, world!");
```

There are four important details to notice here:
- Rust style is to indent with four spaces, not a tab.
- `println!` calls a Rust macro. If it had called a function instead, it would be entered as `println` (without the `!`). 

## Hello, Cargo!
Cargo is Rust's build system and package manager, it handles things like building your code, downloading dependencies, and building those dependencies.

Our "Hello, world!" program doesn't have any dependencies but for more complex programs we will and starting a project with Cargo will make adding dependencies much easier to do. 

### Creating a Project with Cargo
Let's take a look at how creating a project with Cargo differs from what we did earlier.

```Bash
cargo new hello_cargo
cd hello_cargo
```

The first command creates a new directory and project called *hello_cargo*. Inside the *hello_cargo* directory we see a *Cargo.toml* file and a *src* directory with a *main.rs* file inside. Additionally, it initializes a new git repository with a *.gitignore* file. 

> **_NOTE:_** You can change `cargo new` to use a different version control system or no version control system by using the `--vcs` flag.

The *Cargo.toml* file should look something like this:

```Toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
```

The file is in the *TOML(Tom's Obvious, Minimal Language)* format, which is Cargo's configuration format.

`[package]` is a section heading that indicates the following statements are configuring a package. The next three lines set the configuration information Cargo needs to compile your program: The name, the version, and the edition of Rust to use.

`[dependencies]` is the start of a section for you to list any of your project's dependencies. In Rust, packages of code are referred to as *crates*. 

Cargo expects your source files to live inside the *src* directory. The top-level project directory is just for README files, license information , configuration files, and anything else not related to your code. 

### Building and Running a Cargo Project
From the *hello_cargo* directory build your project with the following command:

```Bash
cargo build
   Compiling hello_cargo v0.1.0 (/home/siddharth/src/rust_projects/the_book/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.23s
```

This creates an executable file at *target/debug/hello_cargo* rather than in your current directory. You can run it with the following:

```Bash
./target/debug/hello_cargo
Hello, world!
```

Running `cargo build` for the first time also causes Cargo to create a new file at the top level:*Cargo.lock*. This file keeps track of the exact versions of dependencies in your project. You won't ever need to change this file manually, Cargo manages it for you.

Instead of building with `cargo build` and running it with `./target/debug/hello_cargo` we can use `cargo run` to compile and run the resultant executable all in one command.

```Bash
cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/hello_cargo`
Hello, world!
```

Also notice that there was no output indicating that Cargo was compiling `hello_cargo`. Cargo figured out that the files hadn't changed, so it didn't rebuild but just ran the binary. If you had modified the source code you would have seen output similar to this:

```Bash
cargo run
   Compiling hello_cargo v0.1.0 (/home/siddharth/src/rust_projects/the_book/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.09s
     Running `target/debug/hello_cargo`
Hello, Cargo!
```

Cargo also provides a command called `cargo check` which quickly checks your code to make sure it compiles but doesn't produce an executable. `cargo check` is much faster than cargo build because it skips the step of producing an executable. As such, many Rust programmers use `cargo check` periodically as they write their program to make sure it compiles. Then they run `cargo build` when they're ready to use the executable.

### Building for Release
When your project is finally ready for release, you can use `cargo build --release` to compile it with optimizations. The executable will be created in *target/release* instead of *target/debug*. The optimizations make your Rust code faster but lengthens the time it takes for your program to compile. 

### Cargo as Convention
With simple projects, Cargo doesn't provide a lot of value over using `rustc`, but it will prove its worth as your programs become more intricate. Once programs grow to multiple files or you need a dependency, it's much easier to let Cargo coordinate the build. 

For more information about Cargo, check out its [documentation](https://doc.rust-lang.org/cargo/)

## Summary
In this section we looked at how to:
- Install the latest stable version of Rust using `rustup`
- Update to a newer Rust version
- Open locally installed documentation
- Write and run a "Hello, world!" program using `rustc` directly
- Create and run a new project using the conventions of Cargo

In the next [section](2_programming_a_guessing_game.md) we will build a guessing game program
