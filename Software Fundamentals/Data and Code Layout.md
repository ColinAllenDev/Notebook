## Binary Numbers (Base 2)
Binary numbers are represented with 1's and 0's. Computer scientists sometimes use the prefix "0b" to represent binary numbers.
$$0b1101 = (1\times2^3) + (1\times2^2) + (0\times2^1) + (1\times2^0) = 13$$
## Hexadecimal Numbers (Base 16)
Hexadecimal numbers are repesented with 10 digits (0 - 9) and six letters (A - F). The letters A - F represent decimal values 10 through 15, respectively. A prefix "0x" is used to denote hex numbers in C and C++. This notation is popular because computers generally store data in groups of 8 bits known as *bytes*, and since a single hexadecimal digit represents 4 bits exactly, a *pair* of hex digits represents a byte. For example,$0xFF = 0b11111111 = 255$ is the largest number that can be stored in 8 bits (1 byte).$$0xB052 = (11\times16^3) + (0\times16^2) + (5\times16^1) + (2\times16^0) = (11\times4096) + (0\times256) + (5\times16) + (2\times 1) = 45,138$$
## Fixed and Floating Point Notation
Representing fractions and irrational numbers requires a different format that can express the concept of a decimal point. An early approach take by computer scientists was to used *fixed-point notation*. In this notation, one arbitrarily chooses how many bits will be used to represent the whole part of the number, and the rest of the bits are used to represent the fractional part. As we move left to right (i.e., from the most significant bit to the least significant bit), the magnitude bits represent decreasing powers of two ($..., 16, 8, 4, 2, 1$), while the fractional bits represent decreasing *inverse* powers of two ($\frac{1}{2}, \frac{1}{4}, \frac{1}{8}, \frac{1}{16}, ...$).

The problem with fixed-point notation is that it constrains both the range of magnitudes that can be represented and the amount of precision we can achieve in the fractional part. To solve this, we employ a *floating-point* representation.

In *floating-point notation*, the position of the decimal place is arbitrary and is specified with the help of an exponent. A floating-point number is broken into three parts: the *mantissa*, which contains the relevant digits of the number on both sides of the decimal point, the *exponent*, which indicates where in that string of digits the decimal point lies, and a *sign bit*, which of course indicates whether the value is positive or negative. 

The most common way to lay out these three components in memory is defined by the IEEE-754 standard. It states that a 32-bit floating-point number will be represented with the sign in the most significant bit, followed by 8 bits of exponent and finally 23 bits of mantissa.

The value $v$ represented by a sign bit $s$, en exponent $e$ and a mantissa $m$ is: $$v = s \times 2^{(e - 127)} \times (1 + m)$$
## Primitive Data Types
C and C++ provide a number of primitive data types. The C and C++ standards provide guidelines on the relative sizes and signedness of these data types, but each compiler is free to define the types slightly differently in order to provide maxium performance on the target hardware.
- `char`. A *char* is usually 8 bits and is genreally large enough to hold an ASCII or UTF-8 character. Some compilers define *char* to be signed, while others use unsigned *chars* by default.
- `int`. An *int* is supposed to hold a signed integer value that is the most efficient size for the target platform; it is usually defined to be 32 bits wide on a 32-bit CPU architecture and 64 bits wide on a 64-bit architecture. Compiler options and operating system type also can factor into the size of an *integer*.
- `short`. A *short* is intended to be smaller than an *int* and is 16 bits on many machines.
- `long`. A *long* is as large or larger than an *int* nd may be 32 to 64 bits wide, or even wider, (again depending on CPU architecture, compiler options and the target OS).
- `float`. On most modern compilers, a *float* is a 32-bit IEEE-754 floating-point value.
- `double`. A double is a double-precision (i.e., 64-bit) IEEE-754 floating-point value.
- `bool`. A *bool* is a true/false value. The size of a *bool* varies widely across different compilers and hardware achitectures.

## Declarations, Definitions and Linkage
A C or C++ program is comprised of *translation units*. The compiler takes one .cpp file at a time, and for each one it generates an output file called an object file (.o or .obj). A .cpp file is the smallest unit of translation operated on by the compiler; hence, the name "translation unit".

An object file contains not only the compiled machine code for all of the functions defined in the .cpp file, but also all of its global and static variables. In addition, an object file may contain *unresolved references* to functions and global variables defined in *other* .cpp files.
The compiler only operates on one translation unit at a time, so whenever it encounters a reference to an external global variable or function, it must "go on faith" and assume that the entity in question really exists.

It is the *linkers* job to combine all of the object files into a final executable image. In doing so, the linker reads all of the object files and attempts to resolve all of the unresolved cross-references between them. If it is successfull, an executable image is generated containing all of the functions, global variables, and static variables, with all cross-translation-unit references properly resolved.

The linker's primary job is to resolve external references, and in this capacity it can generate only two kinds of errors:
1. The target of an *extern* reference might not be found, in which case the linker generates an "unresolved symbol" error.
2. The linker might find more than one variable or function with the same name, in which case it generates a "multiply defined symbol" error.

### Declaration versus Definition
In the C and C++ languages, variables and functions must be *declared* and *defined* before they can be used. It is important to understand the difference between a declaration and a definition in C and C++.
- A *declaration* is a description of a data object or function. It provides the compiler with the *name* of the entity and its *data type* or *function signature* (i.e., return type and argument type(s))
- A *definition* describes a unique region of memory in the program. This memory might contain a variable, an instance of a struct or class or the machine code of a function.
In other words, a *declaration* is a *reference to an entity*, while a *definition* is an *entity itself*. A definition is always a declaration, but the reverse is not always the case - it is possible to write a pure declaration in C and C++ and that is not a definition.

Functions are *defined* by writing the body of the function immediatley after the signature, enclosed by curly braces. A pure declaration can be provided for a function so that it can be used in other translation units (or later in the same translation unit). This is done by writing a function signature followed by a semicolon, with an optional prefix of `extern`. Same basic concepts apply for variables and instances of classes and structs.
```cpp
// All of these are variable definitions:
U32 gGlobalInterger = 5;
F32 gGlobalFloatArray[16];
MyClass gGlobalInstance;

// All of these are pure declarations:
extern U32 gGlobalInteger;
extern F32 gGlobalFloatArray[16];
extern MyClass gGlobalInstance;
```

> Note: Any particular data object or function in a C/C++ program can have multiple identical *declarations*, but each can have only one *definition*. If two or more identical definitions exist in a single translation unit, the compiler will notice that multiple entities have the same name and flag an error. If they exist in *different* translation units, the *linker* will provide a "multiply defined symbol" error when it tries to resolve the cross-reference. 

### Inlining
It is usally dangerous to place *definitions* in header files. If a header file containing a definition is included into more than one .cpp file, it's a sure-fire way of generating a "multiply defined symbol" linker error.

Inline function definitions are an exception to this rule, because each invocation of an inline function gives rise to a brand new copy of that function's machine code, embedded directly into the calling function. In fact, inline function definitions *must* be placed in header files if they are to be used in more than one translation unit.

The `inline` keyword is really just a hint to the compiler. It does a cost/benefit analysis of each inline function, weighing the size of the function's code versus the potential performance benefits of inlining it, and the compiler gets the final say as to whether the function will really be inlined or not. 

> Note: Some compilers provide syntax like `__forceinline`, allowing the programmer to bypass the compiler's cost/benefit analysis and control function inlining directly.

### Templates and Header Files 
The definition of a templated class or function must be visible to the compiler across all translation units in which it is used. As such, if you want a template to be usable in more than one translation unit, the template must be placed into a header file (just as inline function definitions must be). 

The declaration and definition of a template are therefore inseperable: you cannot declare templated functions or classes in a header but "hide" their definitions inside a .cpp file, because doing so would render those definitions invisible within any other .cpp file that includes that header.

### Linkage
Every definition in C/C++ has a property known as *linkage*. A definition with *external linkage* is visible to and can be reference by translation units other than the one in which it appears. A definition with *internal linkage* can only be "seen" inside the translation unit in which it appears and thus cannot be references by other translation units.

We call this property *linkage* because it dictates whether or not the linker is permitted to cross-reference the entity in question. So, in a sense, linkage is the translation unit's equivalent of the `public:` and `private:` keywords in C++ class definitions.

By default, definitions have *external* linkage. The `static` keyword is used to change a definition's linkage to internal. Note that two or more identical `static` definitions in two or more different .cpp files are considered to be *distinct entities* by the linker (just as if they had been given different names), so they will *not* generate a "multiply defined symbol" error.

```cpp
// foo.cpp //
// This variable can be used by other .cpp files (external linkage)
U32 gExternalVariable;
// This variable is only usable within foo.cpp (internal linkage)
static U32 gInternalVariable;
// This function can be called from other .cpp files (external linkage)
void externalFunction() {}
// This function can only be called from within foo.cpp
static void internalFunction() {}
// bar.cpp //
// This declaration grants access to foo.cpp's variable.
extern U32 gExternalVariable;
// This 'gInternalVariable' is distinct from the one defined in foo.cpp
static U32 gInternalVariable;
// This function is distinct from foo.cpp's version
static void internalFunction()
// Linker Error: "Multiply Defined Symbol"
void externalFunction() {}
```

> Note: In general it is good to assume that declarations *always* have internal linkage - there is no way to cross-reference a single declaration in multiple .cpp files. (If we put a declaration in a header file, then multiple .cpp files can "see" that declaration, but they are in effect each getting a distinct *copy* of the declaration, and each copy has internal linkage within that translation unit.)

This leads us to the real reason why inline function definitions are permitted in header files: it is because inline functions have *internal linkage* by default, just as if they had been declared *static*. If multiple .cpp files include a header containing an inline function definition, each translation unit gets a private copy of that function's body, and no "multiply defined symbol" errors are generated. The linker sees each copy as a distinct entity.

