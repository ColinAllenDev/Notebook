## Anatomy of a Computer
The simplest computer consists of a *central processing unit (CPU)* and a bank of *memory*, connected to one another on a circuit board called the *motherboard* via one or more *buses*, and connected to external peripheral devices by means of a set of *I/O ports* and or *expansion slots*. This basic design is reffered to as the *von neumann architecture*.
### CPU
The CPU is the "brains" of the computer. It consists of the following components:
- an *arithmetic/log unit (ALU)* for performing integer arithmetic and bit shifting.
- a *floating-point unit (FPU)* for doing floating-point arithmetic (typically using IEEE754 standard representation).
- virtually all modern CPUs also contain a *vector processing unit (VPU)* which is capable of performing floating-point and integer operations on multiple data items in parallel.
- a *memory controller (MC)* or *memory management unit (MMU)* for interfacing with on-chip and off-chip memory devices.
- a bank of *registers* which acts as temporary storage during calculations (among other things).
- a *control unit (CU)* for decoding and dispatching machine language instructions to other components on the chip, and routing data between them.
All these components are driven by a periodic square wave signal known as the *clock*. The frequency of the clock determines the rate at which the CPU performs operations such as executing instructions or performing arithmetic.
#### Registers
In order to maximize performance, an ALU or FPU can usually only perform calculations on data that exists in special high-speed memoru cells called *registers*. Registers are typically physically seperate from the computer's main memory, located on-chip and in close promixity to the components that access them. They're usually implemented using fast, high-cost multi-ported static RAM or SRAM. Because registers aren't part of main memory, they typically don't have addresses but they do have names. These could be as simple as `R0, R1, R2`.

Some of the registers in a CPU are designed to be used for general calculations. They're appropreitally called *general-purpose registers (GPR)*. Every CPU also contains a number of *special-purpose registers (SPR)*. These include:
- the *instruction pointer (IP)*
	Contains the address of the currently-executing instruction in a machine language program.
- the *stack pointer (SP)*
	Contains the address of the top of the program's call stack. The stack can grow up or down in terms of memory addresses. In the case it grows downward, a data item may be *pushed* onto the stack by subtracting the size of the item from the value of the stack pointer, and then writing the item at the address pointed to be the stack pointer. Likewise, an item can be *popped* off the stack by reading it from the address pointed to be the stack pointer, and then adding its size to the stack pointer.
- the *base pointer (BP)*
	Contains the base address of the current function's *stack frame* on the call stack. Stack-allocated variables occupy a range of memory addresses at a unique offset from the base pointer. Such variables can be located in memory by simply subtracting its unique offset from the address stored in the base pointer (assuming the stack grows down).
- the *status register*
	Contains bits that reflect the results of the most-recent ALU operation. 
