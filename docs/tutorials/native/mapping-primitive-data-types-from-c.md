---
type: tutorial
layout: tutorial
title:  "Mapping Primitive Data Types from C"
description: "Primitive Data types from C and how they look in Kotlin/Native"
authors: Eugene Petrenko 
date: 2019-04-15
showAuthorInfo: false
issue: EVAN-5343
---

In this tutorial, we learn what C data types are visible in Kotlin/Native and vice versa. We will: 
- See what [Data Types are in C Language](#types-in-c-language)
- Create a [tiny C Library](#an-example-c-library) that uses those types in exports
- [Inspect Generated Kotlin APIs from a C library](#inspecting-generated-kotlin-apis-for-a-c-library)
- Find how [Primitive Types in Kotlin](#primitive-types-in-kotlin) are mapped to C

## Types in C Language

What types do we have in the C language? Let's first list all of them. I have used the
[C data types](https://en.wikipedia.org/wiki/C_data_types) article from Wikipedia as a basis.
There are following types in the C programming language:
- basic types `char, int, float, double` with modifiers `signed, unsigned, short, long` 
- structures, unions, arrays
- pointers
- function pointers

There are also more specific types:
- boolean type (from [C99](https://en.wikipedia.org/wiki/C99))
- `size_t` and `ptrdiff_t` (also `ssize_t`)
- fixed width integer types, e.g., `int32_t` or `uint64_t` (from [C99](https://en.wikipedia.org/wiki/C99))

There are also the following type qualifiers in the C language: `const`, `volatile`, `restruct`, `atomic`.

The best way to see what C data types are visible in Kotlin is to try it

## An Example C Library

We create a `lib.h` file to see how C functions are mapped into Kotlin:
<div class="sample" markdown="1" mode="c" theme="idea" data-highlight-only="1" auto-indent="false">

```c
#ifndef LIB2_H_INCLUDED
#define LIB2_H_INCLUDED

void ints(char c, short d, int e, long f);
void uints(unsigned char c, unsigned short d, unsigned int e, unsigned long f);
void doubles(float a, double b);

#endif
```
</div>

The file is missing the `extern "C"` block, which is not needed for our example, but may be 
necessary if we use C++ and overloaded functions. The 
[C++ compatibility](https://stackoverflow.com/questions/1041866/what-is-the-effect-of-extern-c-in-c)
thread contains more details on this.

For every set of `.h` files,
we will be using the `cinterop` [C Libraries](/docs/reference/native/c_interop.html)
from Kotlin/Native to generate a Kotlin/Native library,
or `.klib`. The generated library will bridge calls from Kotlin/Native to C. It includes
respective Kotlin declarations for the definitions form the `.h` files.
It is only necessary to have a `.h` file to run the `cinterop` tool. And we do not need to create a 
`lib.c` file, unless we want to compile and run the example.
More details on this are covered in the [C Libraries](/docs/reference/native/c_interop.html) page. It is enough for
the tutorial to create the `lib.def` file with the following content:
<div class="sample" markdown="1" mode="c" theme="idea" data-highlight-only="1" auto-indent="false">

```c
headers = lib.h
```
</div>

We may include all declarations directly into the `.def` file after a `---` separator.
It can be helpful to include macros or other C defines into the code generated by the `cinterop` tool.
Method bodies are compiled and fully included into the binary too. Let's use
that feature to have a runnable example without a need for a C compiler.
To implement that, we need to add implementations to the C functions from the `lib.h` file,
and place these functions into a `.def` file.
We will have the following `interop.def` result:
<div class="sample" markdown="1" mode="c" theme="idea" data-highlight-only="1" auto-indent="false">

```c

---

void ints(char c, short d, int e, long f) { }
void uints(unsigned char c, unsigned short d, unsigned int e, unsigned long f) { }
void doubles(float a, double b) { }
```
</div>

The `interop.def` file is enough to compile and run the application or open it in an IDE.
Now it is time to create project files, open the project in
[IntelliJ IDEA](https://jetbrains.com/idea) and run it. 

## Inspecting Generated Kotlin APIs for a C library

[[include pages-includes/docs/tutorials/native/mapping-primitive-data-types-gradle.md]]

Let's create a `src/nativeMain/kotlin/hello.kt` stub file with the following content
to see how C primitive type declarations are visible from Kotlin:

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
import interop.*

fun main() {
  println("Hello Kotlin/Native!")
  
  ints(/* fix me*/)
  uints(/* fix me*/)
  doubles(/* fix me*/)
}
```
</div>

Now we are ready to
[open the project in IntelliJ IDEA](using-intellij-idea.html)
and to see how to fix the example project. While doing that,
we'll examine how C primitive types are mapped into Kotlin/Native.

## Primitive Types in Kotlin

With the help of IntelliJ IDEA's _Goto Declaration_ or
compiler errors we see the following generated API for our C functions:

<div class="sample" markdown="1" theme="idea" data-highlight-only="1" auto-indent="false">

```kotlin
fun ints(c: Byte, d: Short, e: Int, f: Long)
fun uints(c: UByte, d: UShort, e: UInt, f: ULong)
fun doubles(a: Float, b: Double)
```
</div>

C types are mapped in the way we would expect, note that `char` type is mapped to `kotlin.Byte` 
as it is usually an 8-bit signed value.

| C | Kotlin |
|---|--------|
| char  |  kotlin.Byte |
| unsigned char  |  kotlin.UByte |
| short |  kotlin.Short |
| unsigned short |  kotlin.UShort |
| int   |  kotlin.Int |
| unsigned int   |  kotlin.UInt |
| long long  |  kotlin.Long |
| unsigned long long |  kotlin.ULong |
| float |  kotlin.Float |
| double | kotlin.Double |
{:.zebra}


## Fixing the Code

We've seen all definitions and it is the time to fix the code.
Let's run the `runDebugExecutableNative` Gradle task [in IDE](using-intellij-idea.html)
or use the following command to run the code:
[[include pages-includes/docs/tutorials/native/runDebugExecutableNative.md]]

The final code in the `hello.kt` file may look like that:
 
<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
import interop.*

fun main() {
  println("Hello Kotlin/Native!")
  
  ints(1, 2, 3, 4)
  uints(5, 6, 7, 8)
  doubles(9.0f, 10.0)
}
```
</div>

## Next Steps

We will continue to explore more complicated C language types and their representation in Kotlin/Native
in the next tutorials:
- [Mapping Struct and Union Types from C](mapping-struct-union-types-from-c.html)
- [Mapping Function Pointers from C](mapping-function-pointers-from-c.html)
- [Mapping Strings from C](mapping-strings-from-c.html)

The [C Interop documentation](/docs/reference/native/c_interop.html)
documentation covers more advanced scenarios of the interop.