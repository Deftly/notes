# Concurrency in Go

<!--toc:start-->
- [Concurrency in Go](#concurrency-in-go)
  - [When to Use Concurrency](#when-to-use-concurrency)
  - [Goroutines](#goroutines)
  - [Channels](#channels)
    - [Reading, Writing, and Buffering](#reading-writing-and-buffering)
    - [`for-range` and Channels](#for-range-and-channels)
    - [Closing a Channel](#closing-a-channel)
    - [How Channels Behave](#how-channels-behave)
  - [`select`](#select)
  - [Concurrency Practices and Patterns](#concurrency-practices-and-patterns)
    - [Keep Your APIs Concurrency-Free](#keep-your-apis-concurrency-free)
    - [Goroutines, for Loops, and Varying Variables](#goroutines-for-loops-and-varying-variables)
    - [Always Clean Up Your Goroutines](#always-clean-up-your-goroutines)
    - [The Done Channel Pattern](#the-done-channel-pattern)
    - [Using a Cancel Function to Terminate a Goroutine](#using-a-cancel-function-to-terminate-a-goroutine)
    - [When to Use Buffered and Unbuffered Channels](#when-to-use-buffered-and-unbuffered-channels)
    - [Backpressure](#backpressure)
    - [Turning Off a case in a `select`](#turning-off-a-case-in-a-select)
    - [How to Time Out Code](#how-to-time-out-code)
    - [Using WaitGroups](#using-waitgroups)
    - [Running Code Exactly Once](#running-code-exactly-once)
    - [Putting Our Concurrent Tools Together](#putting-our-concurrent-tools-together)
  - [When to Use Mutexes Instead of Channels](#when-to-use-mutexes-instead-of-channels)
  - [Atomics, You Probably Don't Need These](#atomics-you-probably-dont-need-these)
  - [Where to Learn More About Concurrency](#where-to-learn-more-about-concurrency)
  - [Wrapping Up](#wrapping-up)
<!--toc:end-->

## When to Use Concurrency

## Goroutines 

## Channels 

### Reading, Writing, and Buffering

### `for-range` and Channels 

### Closing a Channel 

### How Channels Behave

## `select`

## Concurrency Practices and Patterns

### Keep Your APIs Concurrency-Free 

### Goroutines, for Loops, and Varying Variables

### Always Clean Up Your Goroutines

### The Done Channel Pattern 

### Using a Cancel Function to Terminate a Goroutine

### When to Use Buffered and Unbuffered Channels

### Backpressure 

### Turning Off a case in a `select`

### How to Time Out Code 

### Using WaitGroups 

### Running Code Exactly Once 

### Putting Our Concurrent Tools Together

## When to Use Mutexes Instead of Channels 

## Atomics, You Probably Don't Need These 

## Where to Learn More About Concurrency

## Wrapping Up
