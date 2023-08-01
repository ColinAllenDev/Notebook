A program written in C or C++ stores its data in a number of different places in memory. In order to understand how storage is allocated and how the various types of C/C++ variables work, it is important to understand the memory layout of a C/C++ program.

## Executable Image
When a C/C++ program is built, the linker creates an *executable file*. Most UNIX-like operating systems, including many game consoles, employ a popular executable file format called *executable and linking format* (ELF).

The executable file contains a partial *image* of the program as it will exist in memory when it runs. (The image is considered a *partial* image because the program generally allocates memory at runtime in addition to the memory laid out in its executable image).

The executable image is divided into contiguous blocks called *segments* or *sections*. The image is usually comprised of at least the following four segments:
1. *Text segment*. Sometimes called the *code segment*, this block contains executable machine code for all functions defined by the program.
2. *Data segment*. This segment contains all *initialized* global and static variables. The memory needed for each global variable is laid out exactly as it will appear when the program is run, and the proper initial values are all filled in. So when the executable file is loaded into memory, the initialized global and static variables are ready to go.
3. *BSS segment*. "BSS" stands for "block started by symbol". This segment contains all of the *uninitialized* global and static variables defined by the program. The C and C++ languages explicitly define the initial value of any uninitialized global or static variable to be zero. But rather than storing a potentially very large block of zeros in the BSS section, the linker simply stores a *count* of how many zero bytes are required to account for all of the uninitialized globals and statics in the segment. When the executable is loaded into memory, the operating system reserves the requested number of bytes for the BSS section and fills it with zeros prior to calling the program's entry point.
4. *Read-only data segment*. Sometimes called the *rodata* segment, this segment contains any read-only (constant, defined with `const`) global data defined by the program. Note that integer constants are often used as *manifest constants* by the compiler, meaning that they are inserted directly into the machine code whenever they are used. Such constants occupy storage in the text segment, but they are not present in the read-only data segment.

Global variables (variables defined at *file scope* outside any function or class declaration) are stored in either the data or BSS segments, depending on whether or not they have been initialized. An unitialized global will be allocated and initialized to zero by the operating system, based on the specifications given in the BSS segment, because it has not been initialized by the programmer.

The `static` keyword can also be used to declare a global variable *within a function*. A function-static variable is *lexically scoped* to the function in which it is declared (i.e., the variable's name can only be "seen" inside the function). It is initialized the first time the function is called (rather than before `main()` is called, as with file-scope statics). But in terms of memory layout in the executable image, a function-static variable acts identically to a file-static global variable - it is stored in either the data or BSS segment based on whether or not it has been initialized.

```cpp
void readBook(U32 book) {
	static U32 sBooks = 5; // data segment
	static U32 sRead; // BSS segment
}
```

## Program Stack
When an executable program is loaded into memory and run, the operating system reserves an area of memory for the *program stack*. Whenever a function is called, a contiguous area of stack memory is push onto the stack - we call this block of memory a *stack frame*.

If function `a()` calls another function `b()`, a new stack from for `b()` is pushed on top of `a()`'s frame. When `b()` returns, its stack frame is popped, and execution continues wherever `a()` left off. 

A stack from stores three kinds of data:
1. It stores *return addresses* of the calling function so that execution may continue in the calling function when the called function returns.
2. The contents of all relevant *CPU registers* are saved in the stack frame. This allows the new function to use the registers without fear of overwriting data needed by the calling function. Upon return to the calling function, the state of the registers is restored so that execution of the calling function may resume. The return value of the called function, if any, is usually left in a specific register so that the calling function can retrieve it, but the other registers are restored to their original values.
3. The stack frame also contains all *local variables* declared by the function; these are also known as *automatic variables*. This allows each distinct function invocation to maintain its own private copy of every local variable, even when a function calls itself recursively. 

Pushing and popping stack frames is usally implemented by adjusting the value of a single register in the CPU, known as the *stack pointer*. When a function containing automatic variables returns, its stack frame is abandoned and all autmoatic variables in the function should be treated as if they no longer exist. Technically, the memory occupied by those variables is still there in the abandoned stack frame - but that memory will very likely be overwritten as soon as another function is called.

A common error involves returning the address of a local variable, like this:
```cpp
U32* getAnInteger() {
	U32 anInteger = 42;
	return &anInteger;
}
```
You *might* get away with this if you use the returned pointer immediately and don't call any other functions in the intermin. But more often than not, this kind of code will crash.

## Dynamic Allocation Heap
When globals and statics are allocated within the executable image, as defined by the data and BSS segments of the executable file, as well as when the locals are allocated on the program stack, both of these kinds of storage are considered *statically* defined, meaning the size and layout of the memory is known when the program is compiled and linked. However, a program's memory requirements are often not fully known at compile time. A program usually needs to allocate additional memory *dynamically*.

To allow for dynamic allocation, the operating system maintains a block of memory for each running process from which memory can be allocated by calling `malloc` (or an OS-specific function like `HeapAlloc()` under Windows) and later returned for reuse by the process at some future time by calling `free()` (or `HeapFree()`). This memory block is known as *heap memory*, or the *free store*. When we allocate memory dynamically, we can sometimes say that this memory resides *on the heap*.

> Note: In C++, the global `new` and `delete` operators are used to allocate and free memory to and from the free store. Be wary, however - individual classes may overload these operators to allocate memory in custom ways, and even the *global* `new` and `delete` operators can be overloaded, so you cannot simply assume that `new` is always allocating from the global heap.

## Member Variables
C structs and C++ classes allow variables to be grouped into logical units. It's important to remember that a class or struct *declaration* allocates no memory. It is merely a description of the layout of the data - a cookie cutter which can be used to stamp out *instances* of that struct or class later on. Once a struct or class has been declared, it can be allocated (defined) in any of the ways that a primitive data type can be allocated:
- as an atomic variable on the program stack
```cpp
void someFunction() {
	Foo localFoo;
}
```
- as a global, file-static or function-static
```cpp
Foo gFoo;
static Foo sFoo;
void someFunction() {
	static Foo sLocalFoo;
}
```
- dynamically allocated from the heap
```cpp
Foo* gpFoo = nullptr; // global pointer to an instance of Foo
void someFunction() {
	// Allocate a foo instance from the heap
	gpFoo = new Foo;
	// Allocate another Foo, assign to a local pointer
	Foo* pAnotherFoo = new Foo;
	// Allocate a POINTER to a Foo from the heap
	Foo** ppFoo = new Foo*;
	(*ppFoo) = pAnotherFoo;
}
```
In the instance of allocating a class or struct from the heap, the pointer or reference variable used to hold the address of the data can be itself allocated as an automatic, global, static or even dynamically.

## Class-Static Members
As we've seen, the `static` keyword has many different meaninigs depending on context:
- When used at file scope, `static` means "restrict the visibility of this variable or function so it can only be seen inside this .cpp file."
- When used at function scope, `static` means "this variable is a global, not an automatic, but it can only be seen inside this function."
- When used inside a `struct` or `class` declaration, `static` means "this variable is not a regular member variable, but instead acts just like a global."

Notice that when `static` is used inside a class declaration, it does not control the *visibility* of the variable (as it does when used at file scope) - rather, it differentiates between regular per-instance member variables and per-class variables that act like globals. The *visibility* of a class-static variable is determined by the use of `public:`, `private:`, and `protected:` keywords in the class declaration.

Class-static variables are automatically included within the namespace of the class or struct in which they are declared. So the name of the class or struct must be used to disambiguate the varaible whenever it is used outside that class or struct.

Like an `extern` declaration for a regular global variable, the declaration of a class-static variable within a class *allocates no memory*. The memory for the class-static variable must be defined in a .cpp file.
```cpp
// foo.h //
class Foo {
public:
	static F32 sClassStatic; // Allocates no memory!
};
// foo.cpp //
F32 Foo::sClassStatic = -1.0f; // Define memory and initialize
```

### Object Layout in Memory
It's useful to visualize the memory layout of your classes and structs. This can typically be done by drawing a box for the struct or class, with horizontal lines seperating data members. On the lefthand side of each box, we can denote the hexadecimal memory address of the beggining of each block. The width of each block should represent the size of each data member.
![[img_objmemlayout.png|300]]
You might imagine that as you layout structs and classes that small data members intersected with large ones will simply be packed together in memory by the compiler as tightly as it can. This, however, is not usually the case. Instead, the compiler will typically leave "holes" in the layout. (Some compilers can be requested not to leave these holes by using a pre-processor directive like `#pragma pack`; but the default behavior is to space out members as seen in the figure below.)
![[img_objmemlayout1.png| 250]]
Why does the compiler leave these "holes"? The reason lies in the fact that every data type has a natural *alignment*, which must be respected in order to permit the CPU to read and write memory effectively. The *alignment* of a data object refers to whether its *address in memory* is a multiple of its *size* (which is generally a power of two):
- An object with 1-byte alignment resides at any memory address.
- An object with 2-byte alignment resides only at even addresses (i.e., addresses whose least significant nibble (typically half a *byte* or *4 bits*) is `0x0`, `0x2`, `0x4`, `0x8`, `0xA`, `0xC`, or `0xE`).
- An object with 4-byte alignment resides only at addresses that are a multiple of four (i.e., addresses whose least significant nibble is `0x0`, `0x4`, `0x8` or `0xC`)
- An object with 16-byte alignment resides only at addresses that are a multiple of 15 (i.e., addresses whose least significant nibble is `0x0`)

Alignment is important because many modern processors can actually only read and write properly aligned blocks of data. For example, if a program requests that a 32-bit (4-byte) integer be read from address `0x6A341174`, the memory controller will load the data happily because the address is 4-byte aligned (in this case, the least significant nibble is `0x4`). However, if a request is made to load a 32-bit integer from address `0x6A341173`, the memory controller now has to read *two* 4-byte blocks: the one at `0x6A341170` and the one at `0x6A341174`. It must then mask and shift the two parts of the 32-bit integer and logically OR them together into the desination register on the CPU.

A good rule of thumb is that a data type should be aligned to a boundary equal to the width of the data type in bytes. For example, 32-bit values generally have a 4-byte alignment requirement, 16-bit values have a 2-byte alignment, and 8-bit values can be stored at any address (1-byte aligned). By considering the layout of the members of your data structures, you can elimitate these "holes" that are wasting space by sorting these members by their alignment.  

```cpp
struct EfficientPadding {
	U32 mU1; // 32 bits (4-byte aligned)
	U32 mU2; // 32 bits (4-byte aligned)
	U32 mU3; // 32 bits (4-byte aligned)
	U32 mU4; // 32 bits (4-byte aligned)
	bool mB1; // 8 bits (1-byte aligned)
	bool mB2; // 8 bits (1-byte aligned)
}
```
