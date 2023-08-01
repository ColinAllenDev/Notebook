## Exception Handling
*Exception handling* allows the function that detects a problem to communicate the error to the rest of the code without knowing anything about which function might handle the error. When an exception is thrown, relevant information about the error is placed into a data object of the programmer's choice known as an *exception object*. The call stack is then automatically unwound, in search of a calling function that has wrapped it's call in a *try-catch block*. If a *try-catch* block is found, the exception object is matched against all possible *catch* clauses, and if a match is found, the corresponding *catch*'s code block is executed.

## Assertions
An *assertion* is a line of code that checks an expression. If the expression evaluates to true, nothing happens. But if the expression evaluates to false, the program is stopped, a message is printed and the debugger is invoked if possible.

Assertions check programmer's assumptions. They check the code when it is first written to ensure that it is functioning properly. They also ensure that the original assumptions continue to hold for long periods of time, even when the code around them is constantly changing.

**Implementation**
```cpp
#if ASSERTIONS_ENABLED
// Define some inline assembly that causes a break into the debugger (this will be different on each target CPU)
#define debugBreak() asm { int 3 }

// Check the expression and fail if it is false
#define ASSERT(expr) \
	if (expr) {} \
	else \
	{ \
		reportAssertionFailure(#expr, __FILE__, __LINE__); \
		debugBreak(); \
	} 
#else
#define ASSERT(expr) // evaluates to nothing
#endif
```

**Compile-Time Assertions (a.k.a Static Assertations)**
One weakness of assertions is that the conditions encoded within them are only checked at *runtime*. We have to run the program, *and* the code path in question must actually *execute*, in order for an assertion's condition to be checked.

*static_assert()* is a macro defined in C++11's standard library. The macro places a declaration into your code that (a) is legal at file scope, (b) evaluates the desired expression into your code at compile time rather than runtime, and (c) produces a compile error if and only if the expression is false.  [More info.](http://www.pixelbeat.org/programming/gcc/static_assert.html)

**Implementation**
```cpp
#ifdef __cpluscplus
	#if __cplusplus >= 201103L
		#define STATIC_ASSERT(expr) \
			static_assert(expr, "static assert failed: " #expr)
	#else
		// Declare a template but only define the true case (via specialization)
		template<bool> class TStaticAssert;
		template<> class TStaticAssert<true> {};
		#define _ASSERT_GLUE(a, b) a ## b
		#define ASSERT_GLUE(a, b) _ASSERT_GLUE(a, b)
		#define STATIC_ASSERT(expr) \
			enum { ASSERT_GLUE(g_assert_fail_, __LINE__) \
						= sizeof(TStaticAssert<!!(expr))}
```