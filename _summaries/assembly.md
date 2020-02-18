---
title: "Programming from the ground up"
author: "Me"
date: "Fbruary 17, 2020"
output: html_document
---

<style type="text/css">

body{ /* Normal  */
      font-size: 12px;
  }
td {  /* Table  */
  font-size: 12px;
}
h1.title {
  font-size: 38px;
  color: DarkRed;
}
h1 { /* Header 1 */
  font-size: 28px;
  color: DarkRed;
}
h2 { /* Header 2 */
    font-size: 22px;
  color: DarkRed;
}
h3 { /* Header 3 */
  font-size: 18px;
  font-family: "Times New Roman", Times, serif;
  color: DarkRed;
}
code.r{ /* Code block */
    font-size: 12px;
}
pre { /* Code block - determines code spacing between lines */
    font-size: 14px;
}
</style>


# Programming from the ground up
{:.no_toc}

This is a resumee of the book “Programming from the ground up”. For this resumee, I asked myself different questions and used
the book as well as other resources to answer them.

The chapters are the following :

1. TOC
{:toc}
{:style="color:black; font-size: 150%;"}

# Introduction

What is assembly language ?
{:style="color:DarkRed; font-size: 170%;"}
There are 3 kinds of languages :
- Machine language : what the computer sees and uses (sequence of numbers) → Specific to the processor.
- Assembly language : same as machine language (also specific to the processor), but the numbers are replaced by letters. Easier to use for humans
- High-level language : closer to natural language. The purpose is to make programming easier. Not specific to the computer (portable across multiple systems).

The syntax of assembly instructions is the following:\\
Operation destination, source\\
Example :

| Assembly | Machine language (Intel) |
| ---: | ---:|
| mov epb, esp | 89 e5 |
| sub esp, 0x8 | 83 ec 08 |
{:.table-striped}
		
→ This example transfers the value of esp into ebp, and subtracts 8 (0x8) to esp. The result is stored into esp. We will soon see what esp and ebp are.

Assembly language is converted into executable machine code by a utility program referred to as an **assembler**.
High-level languages are converted to machine language by a compiler.

Why learn assembly ?
{:style="color:DarkRed; font-size: 170%;"}
Assembly language is very low level and close to the processor → assembly is great for speed optimization.
Also, understanding assembly language allows to fully understand what a program does and therefore is very useful for reverse engineering tasks.


# Computer Architecture

What is the modern computer architecture ?
{:style="color:DarkRed; font-size: 170%;"}
Modern computer architecture is based off of the Von Neumann architecture. This latter divides the computer into 2 main parts :

- The memory
- The CPU (Central Processing Unit) or processor

What is x86 ?
{:style="color:DarkRed; font-size: 170%;"}
x86 is an instruction set (“language”) developed by Intel for the x86 class of processors. In other words, it’s the language a x86 CPU (or processor) speaks.
The vast majority of private computers have x86 CPUs, that only understand x86 assembly language.

What is the memory of a computer and what does it contain ?
{:style="color:DarkRed; font-size: 170%;"}
Everything in a computer lives in the memory (data and programs), in fixed-size storage locations (each location has a number and the same fixed-length size).
The size of a single storage location is called a byte. On x86 processors, a byte is a number between 0 and 255 (1 byte = 8 bits →  2^8 = 256). We can only store a single number in a computer memory storage location (one kind of data). Therefore, the memory contains only numbers.

What is a CPU or processor ?
{:style="color:DarkRed; font-size: 170%;"}
The CPU (or processor) allows us to access the data stored in memory, manipulate, and move it. It reads instructions from the memory (instructions of a program) one at a time and executes them → this is called the **fetch-execute cycle**. To achieve this, the CPU has :

1. a **Program Counter (PC)** : it holds the memory address of the next instruction to be executed (a number) → tells the computer what instruction to process by indicating an address in the memory. The CPU begins by looking at the PC, fetch the number stored at the specified location, and passes it to the instruction decoder.

2. an **instruction decoder** : decodes what the number specified by the PC means (+, -, x, data movement, … and what memory locations are involved in the process).

3. a **data bus** : used by the computer to fetch the memory locations to be used in the calculation. It’s the connection between the CPU and memory (the actual wire that connects them).

4. **Registers** : high-speed memory locations of the processor itself. It’s where the main action happens, used for computation (+, -, x, comparisons, …)

5. an **Arithmetic and logic unit** (ALU) : once the CPU has retrieved all the data it needs, it passes the data and the decoded instruction to the ALU, where the instruction is actually executed. When the computation is done, the results are placed on the data bus and sent to the appropriate location in memory or in a register.

In short, the fetch-execute cycle consists in 3 steps: fetching the instruction from the memory, decoding it, and executing it.

How are numbers converted to bits ?
{:style="color:DarkRed; font-size: 170%;"}
The memory is a numbered sequence of fixed-size storage locations. The number attached to each storage location is called its address. The size of a single storage location is called a byte. On x86 processors, a byte is a number between 0 and 255 (because 1 byte = 8 bits, and a bit can be either 0 or 1 ; it has 2 possible state → 2^8 = 256, but we start from 0 so 0-255).
Example :\\
0 0 0 0 0 0 0 0	= 0 (0\*2^7 + 0\*2^6 + 0\*2^5 + 0\*2^4 + 0\*2^3 + 0\*2^2 + 0\*2^1 + 0\*2^0)\\
1 1 1 1 1 1 1 1	= 255 (1\*2^7 + 1\*2^6 + 1\*2^5 + 1\*2^4 + 1\*2^3 + 1\*2^2 + 1\*2^1 + 1\*2^0)\\
0 0 0 0 0 1 1 1	= 7 (0\*2^7 + 0\*2^6 + 0\*2^5 + 0\*2^4 + 0\*2^3 + 1\*2^2 + 1\*2^1 + 1\*2^0)\\
1 0 1 0 1 0 1 0	= 171 (1\*2^7 + 0\*2^6 + 1\*2^5 + 0\*2^4 + 1\*2^3 + 0\*2^2 + 1\*2^1 + 1\*2^0)

How can a computer use and display text, if the memory only contains numbers ?
{:style="color:DarkRed; font-size: 170%;"}
Specialized hardware (like graphics cards) have special interpretations for each number. For text, the computer uses ASCII code tables to translate the numbers into letters, and vice versa. For example, “A” is represented by the number 65. To print “HELLO”, we would give the computer the sequence of numbers 72, 69, 76, 76, 79.

What if we need numbers larger than 255 ?
{:style="color:DarkRed; font-size: 170%;"}
We can combine bytes. 2 bytes (=16 bits) can be used to represent numbers between 0 and 65’535 (2^16), and so on. Fortunately, the computer does it for us and work with 4 bytes numbers by default.

What is big-endian and little endian ?
{:style="color:DarkRed; font-size: 170%;"}
The x86 architecture is “little endian” → multi-bytes values are written least significant byte first. The least significant byte (smallest power of 2) is placed at the byte with the lowest address. It is the opposite in big endian.\\
Example: let's consider the hexadecimal number 0725 (= 1829 in decimal: 0\*16^3 + 7\*16^2 + 2\*16^1 + 5\*16^0).
This number requires 2 bytes of memory. The most significant byte is 07, and the least significant is 25. If the processor brings the value 0725 from register to memory, it will transfer 25 first to the lower memory address and 07 to the next address.

![test](/_images/endian.png)
{:class="img-responsive"}

How can we represent negative numbers in binary ?
{:style="color:DarkRed; font-size: 170%;"}
We can use the method “2’s complement” : the sign is changed by inverting all the bits and adding one. Example :\\
Start :		0 0 0 1 (represents decimal 1)\\
Invert :		1 1 1 0\\
Add one :	1 1 1 1 (represents decimal -1)

What are registers ?
{:style="color:DarkRed; font-size: 170%;"}
Registers are high-speed memory locations (the “working memory”) of the processor itself.
They keep the contents of numbers that we are manipulating.
On 32-bits processor computers, the registers are 4 bytes long (32 bits). On 64-bits, they are 8 bytes long. The size of a typical register is called a computer’s word size. Old x86 processors have 4 byte words (old x86 are 32 bits processor, recent ones are 64 bits (also called x86-64)). 
Addresses are also 4 bytes (= 1 word) long, and therefore fit into 1 register.
So, a 32 bits x86 processor can access up to 4294967296 (2^32) bytes.
A 64 bits x86 processor can access up to 1.8446744e+19 (2^64) bytes.
This means that we can store addresses the same way we store any other number → the computer can’t tell the difference between a value that is an address, or a value that is a number, or a value that is an ASCII code, … A number becomes an ASCII code when we try to display it, a number becomes an address when we try to look up the byte it points to. Addresses stored in memory are also called pointers, because they point to a different location in memory.

How does a computer know how to interpret a given byte or set of bytes of memory ?
{:style="color:DarkRed; font-size: 170%;"}
The only way the computer knows that a memory location is an instruction is that a special-purpose register called the instruction pointer (EIP) points to them at one point or another. If the instruction pointer points to a memory location, it is loaded as an instruction. Other than that, the computer has no way of knowing the difference between instructions (programs) and other types of data.

What are the registers of the x86 architecture ?
{:style="color:DarkRed; font-size: 170%;"}
The x86 architecture has 8 General-Purpose Registers (GPR), 6 Segment Registers, 1 Flags Register and an Instruction Pointer. 64-bit x86 has additional registers.

The **8 GPRs** are:

1. Accumulator register (AX): used in arithmetic operations
2. Counter register (CX): used in shift/rotate instructions and loops.
3. Data register (DX): used in arithmetic operations and I/O (input/output) operations.
4. Base register (BX): used as a pointer to data.
5. Stack Pointer register (SP): pointer to the top of the stack.
6. Stack Base Pointer register (BP): used to point to the base of the stack.
7. Source Index register (SI): used as a pointer to a source in stream operations.
8. Destination Index register (DI): used as a pointer to a destination in stream operations.
{:style="color:#333; font-size: 150%;"}

The order is important: it is the same order that is used in a push-to-stack operation. These are the registers in 16-bit mode.  In 32-bit mode, the two-letter abbreviations above are prefixed with an 'E' (extended). For example, 'EAX' is the accumulator register as a 32-bit value.
Similarly, in the 64-bit version, the 'E' is replaced with an 'R' (register), so the 64-bit version of 'EAX' is called 'RAX'.

The **6 Segment Registers** are : 

1. Stack Segment (SS): pointer to the stack.
2. Code Segment (CS): pointer to the code.
3. Data Segment (DS): pointer to the data.
4. Extra Segment (ES): pointer to extra data.
5. F Segment (FS): pointer to more extra data ('F' comes after 'E').
6. G Segment (GS): pointer to still more extra data ('G' comes after 'F').
{:style="color:#333; font-size: 150%;"}

Most applications on modern operating systems use a memory model that points nearly all segment registers to the same place. Therefore, their use is not common. However, FS and GS are exceptions to this rule, instead being used to point at thread-specific data.
 
The EFLAGS is a 32-bit register used as a collection of bits representing Boolean values to store the results of operations and the state of the processor.

The names of **EFLAGS bits** are:

0. CF: Carry Flag. Set if the last arithmetic operation carried (addition) or borrowed (subtraction) a bit beyond the size of the register. This is then checked when the operation is followed with an add-with-carry or subtract-with-borrow to deal with values too large for just one register to contain.
2. PF: Parity Flag. Set if the number of set bits in the least significant byte is a multiple of 2.
4. AF: Adjust Flag. Carry of Binary Code Decimal (BCD) numbers arithmetic operations.
6. ZF: Zero Flag. Set if the result of an operation is Zero (0).
7. SF: Sign Flag. Set if the result of an operation is negative.
8. TF: Trap Flag. Set if step by step debugging.
9. IF: Interruption Flag. Set if interrupts are enabled.
10. DF: Direction Flag. Stream direction. If set, string operations will decrement their pointer rather than incrementing it, reading memory backwards.
11. OF: Overflow Flag. Set if signed arithmetic operations result in a value too large for the register to contain.
12 & 13. IOPL: I/O Privilege Level field (2 bits). I/O Privilege Level of the current process.
14. NT: Nested Task flag. Controls chaining of interrupts. Set if the current process is linked to the next process.
16. RF: Resume Flag. Response to debug exceptions.
17. VM: Virtual-8086 Mode. Set if in 8086 compatibility mode.
18. AC: Alignment Check. Set if alignment checking of memory references is done.
19. VIF: Virtual Interrupt Flag. Virtual image of IF.
20. VIP: Virtual Interrupt Pending flag. Set if an interrupt is pending.
21. ID: Identification Flag. Support for CPUID instruction if can be set.
{:style="color:#333; font-size: 150%;"}
 

Finally, the **Instruction pointer** (EIP) register contains the address of the next instruction to be executed if no branching is done.
EIP can only be read through the stack after a call instruction.

What is a stack ?
{:style="color:DarkRed; font-size: 170%;"}
It's a data structure used to store objects. Items can be added using a **push** operation, and retrieved with a **pop** operation. An object added comes to the top of the stack. Items can be removed from the top (*LIFO -> Last In, First Out*) or from the bottom (*FIFO -> First In, First Out*) of the stack.\\
If a stack runs out of memory (i.e if it's full, no more object can be pushed), it will cause a **stack overflow**.

How does the processor access data ?
{:style="color:DarkRed; font-size: 170%;"}
The ways a processor access data are called **addressing modes**. The **6 most important addressing modes** are:

1. Immediate mode: it's the simplest mode, because the data to access is embedded in the instruction itself. For example, to initialize a register to 0, we give it the number 0 instead of giving an address to read the 0 from.

2. Register addressing mode: the instruction contains a register to access, rather than a memory location.

3. Direct addressing mode: the instruction contains the memory address to access. Example: we can ask the computer to load a register with the data at address 2000. It would go directly to byte 2000 and copy the contents into the register.

4. Indexed addressing mode: the instruction contains the memory address to access, but also specifies an index register to offset that address.\\ 
Example: we specify address 2000 and an index register. If the index register contains the number 20, the actual address the data is loaded from would be 2020 (useful to cycle between numbers with an index).\\
On x86 processors, we can also specify a multiplier for the index (allows to access memory a byte at a time, or a word at a time (4 bytes)).\\
Example : if we want to access the 4th byte from location 2000, we would load our index register with 3 (counting start at 0), and set the multiiplier to 1. This would get us location 2003.

5. Indirect addressing mode: the instruction contains a register that contains a pointer to where the data should be accessed.\\
Example: if we specify the %eax register, and that this register contains the value 4, the value contained at memory location 4 would be used.

6. Base-pointer addressing mode: similar to indirect addressing, but we also include a number called the *offset* to add to the register's value before using it for lookup. This mode will be widely used in the book.

# Your First Programs

# All about functions

# Dealing with Files

# Reading and Writing Simple Records

# Developing Robust Programs

# Sharring Functions with Code Libraries

# Intermediate Memory Topics

# Counting Like a Computer

# High-Level Languages

# Optimization

# Moving On from Here
