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
through it. It can then vary between 0 and 2<sup>16</sup> - 1, so 65536 addresses available.

### Memory map

As we said previously, you can and you will have several memory in your system. Global working RAM,
ROM, video dedicated RAM, and so on, but the address space is unique, and the way a subset of the
address space is allocated to a specific memory instance is what we call the _memory map_.

The _memory map_ is a specification that splits the address space in subsets that are dedicated
to a particular block of memory. Following table describes the _memory map_ associated to the
gameboy address bus, but keep in mind that you can easily a more detailled memory map with slot
description, map variation depending on console model.

| Address range | Target                    |
| ------------- | ------------------------- |
| $0000 - $3FFF | ROM bank 00               |
| $4000 - $7FFF | ROM bank 01 - NN          |
| $8000 - $9FFF | VRAM                      |
| $A000 - $BFFF | External RAM              |
| $C000 - $CFFF | WRAM bank 0               |
| $D000 - $DFFF | WRAM bank 1               |
| $E000 - $FDFF | $C000 - $DDFF mirror      |
| $FE00 - $FE9F | OAM                       |
| $FEA0 - $FEFF | Not usable                |
| $FF00 - $FF7F | I/O registers             |
| $FF80 - $FFFE | HRAM                      |
| $FFFF - $FFFF | Interrupt enable register |

Please note the used address notation, as it will be the same through all the book and probably on
most documentation you will find over the web. An address is prefixed by the dollar sign, and consists
in 4 digits, using hexadecimal notation, which represents the effective 16-bit address. Remember, 16-bit,
where 8-bit define a byte, which can be written as a two hexadecimal digits. If you don't get it, google it
and do the math. I recommand having the _memory map_ close to you all the time, using a post it,
a printed sheet, or a browser tab, it is your call.

### Memory banking

## Design

## Testing

## Implementation

