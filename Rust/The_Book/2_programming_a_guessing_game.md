# Programming a Guessing Game
This section introduces a few common Rust concepts by showing you how to use them in a real program. We'll cover `let`, `match`, methods, associated functions, external crates, and more. The following sections will cover these concepts in more detail.

Here's how our program will work: the program will generate a random integer between 1 and 100. It will then prompt the player to enter a guess. After a guess is entered, the program will indicate whether the guess is too low or too high. If the guess is correct, the game will print a congratulatory message and exit.

## Processing a Guess
The first part of the program will be to ask the user for input and check that the input is of the expected form. 

```Rust
use std::io;

fn main() {
    println!("Guess the number!");

    println!("Please input your guess:");

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Falied to read line");

    println!("You guessed: {guess}");
}
```

To obtain user input we have to bring the `io` input/output library into scope. The `io` library comes from the standard library, known as `std`:

```Rust
use std::io;
```

By default, Rust has a set of items defined in the standard library that it brings into the scope of every program. This set is called the *prelude*.

If a type you want isn't in the prelude, you have to bring that type into scope explicitly with a `use` statement.

As we saw previously, the `main` function is the entry point into the program:

```Rust
fn main() {
```

The `fn` syntax declares a new function. The parentheses, `()`, indicate there are no parameters.

`println!` is a macro that prints a string to the screen:

```Rust
println!("Guess the number!");
println!("Please input your guess:")
```

## Storing Values with Variables
Next we'll need a *variable* to store the user input:

```Rust
let mut guess = String::new();
```

In Rust, variables are immutable by default, meaning once we give the variable a value, the value won't change. We'll discuss the concept in detail in [section 3](3_common_programming_concepts.md). To make a variable mutable, we add `mut` before the variable name.

```Rust
let apples = 5; // immutable
let mut bananas = 5; // mutable
```

> **_NOTE:_** The `//` syntax starts a comment that continues till the end of the line.

We now know that `let mut guess` creates a mutable variable named guess. The equal sign(`=`) tells Rust we want to bind something to the variable now. 
