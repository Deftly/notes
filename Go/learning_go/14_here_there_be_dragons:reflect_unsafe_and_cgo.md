# Here There Be Dragons: Reflect, Unsafe, and Cgo

<!--toc:start-->
- [Here There Be Dragons: Reflect, Unsafe, and Cgo](#here-there-be-dragons-reflect-unsafe-and-cgo)
  - [Reflection Lets Us Work with Types at Runtime](#reflection-lets-us-work-with-types-at-runtime)
    - [Types, Kinds, and Values](#types-kinds-and-values)
    - [Making New Values](#making-new-values)
    - [Use Reflection to Check If an Interface's Value is `nil`](#use-reflection-to-check-if-an-interfaces-value-is-nil)
    - [Use Reflection to Write a Data Marshaler](#use-reflection-to-write-a-data-marshaler)
    - [Build Functions with Reflection to Automate Repetitive Tasks](#build-functions-with-reflection-to-automate-repetitive-tasks)
    - [You Can Build Structs with Reflection, but Don't](#you-can-build-structs-with-reflection-but-dont)
    - [Reflection Can't Make Methods](#reflection-cant-make-methods)
    - [Only Use Reflection If It's Worthwhile](#only-use-reflection-if-its-worthwhile)
  - [`unsafe` is Unsafe](#unsafe-is-unsafe)
    - [Use `unsafe` to Convert External Binary Data](#use-unsafe-to-convert-external-binary-data)
    - [`unsafe` Strings and Slices](#unsafe-strings-and-slices)
    - [`unsafe` Tools](#unsafe-tools)
  - [`Cgo` is for Integration, Not Performance](#cgo-is-for-integration-not-performance)
  - [Wrapping Up](#wrapping-up)
<!--toc:end-->

## Reflection Lets Us Work with Types at Runtime 

### Types, Kinds, and Values 

### Making New Values 

### Use Reflection to Check If an Interface's Value is `nil` 

### Use Reflection to Write a Data Marshaler

### Build Functions with Reflection to Automate Repetitive Tasks

### You Can Build Structs with Reflection, but Don't 

### Reflection Can't Make Methods 

### Only Use Reflection If It's Worthwhile 

## `unsafe` is Unsafe 

### Use `unsafe` to Convert External Binary Data 

### `unsafe` Strings and Slices 

### `unsafe` Tools

## `Cgo` is for Integration, Not Performance

## Wrapping Up
