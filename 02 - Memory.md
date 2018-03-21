# Chapter 2 : Memory

TODO : Introductive text.

##

### Address space

Before talking of address space, let's review the fundamental about how memory is managed by
a CPU or any processing unit that which to access to a memory. While investingating over
documentation, lot of concept emerge and don't be affraid to admit it, you have sometime no
clue about what is it about. Opcode, Address bus, instruction set, ALU, ... so many knowledges
that at some point in your learning path you probably have seen, but totally forgot has you
never manipulate them in a real project, or use it in your daily job.

Addressing is one of them, and more generally, how memory is designed in a low level / hardware
point of view. Memory, or memories has you can have several memory in your system, is connected
to the CPU (and yet all other processing unit) through an address bus and a data bus.

When a unit need to read the memory, it put the requested memory address in the address bus. Then
the associated data is put in the data bus that the unit can read. When a unit need to write the
memory, it put the target memory address in the address bus, and the target data in the data bus.
Nothing more, nothing less.

The address space is the range of address that can be reached through your address bus. In case of
the gameboy, the address bus has a 16-bit size, which means we can transit address of 16-bit size
through it. It can then vary between 0 and 2<sup>16</sup> - 1

### Memory banking

### Memory map


## Design

## Testing

## Implementation

