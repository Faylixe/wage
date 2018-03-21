# Chapter 2 : Memory

TODO : Introductive text.

## TODO : Title for memory theory.

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

On some _memory map_ items, you can notice that some target are labelled with "bank" following by an
numeric identifier : this refers to another concept that is _memory banking_.

TODO : Switchable memory bank.

## Design

Alright, now we are a little bit more confortable in how memory works, we can start doing some system
design for it. Since with are using **Java** language, we will think in an oriented object manner,
which is good as we can see a gameboy as a composition of several object, and here is what we need to
represents our overall memory :

- An **AddressBus**
- Various memory banks

But before diving into it, we will define behaviors we need from our memory :

- Read a byte at a given address.
- Write a byte at a given address.

That is quite straightforward, we can write an interface for it :

```java
public interface IMemoryStream {

    byte readByte(int address) throws IllegalAccessException;

    void writeByte(byte value, int address) throws IllegalAccessException;

}
```

Despite how simple the interface is, we already face lot of issues here. First of all the storage aspect,
each memory cell is represented by a **byte** value, which is 8-bit long in _Java_. Since an gameboy address
reach a 8-bit block of data this is perfect, no memory waste, but all primitive types in _Java_ are signed.
We should then keep this fact in mind when we will implements artihmetic operations later. The second main
issues is the address storage type : we went for a **int** here which is 32-bit long. This means that we will
use twice the amount of memory required to store an address since address are 16-bit long for the gameboy.
Why not going for the **short** type which is *16-bit* long ? Well as for the storage aspect, **short**
value are signed, which implies that values can go between -32768 and 32767 or we want an [0, 65536]
address space. In order to ensure a given address is valid regarding of the address space, we then use
an signed **int** which can cover the required range. Another option is to effectively use a **short**
and build the real address on the fly by computing the unsigned value from bits, but since we aims to use
addresses for memory indexing in a time efficient datastructure, this choice wasn't retained.

We now have made some choices, in how we reach the memory. We will continue then by modeling how we will
organizes objects to represents it. As said earlier, we want to represent an address bus, which will be
the central point for accessing memory. Our address bus will receive I/O operation through the **IMemoryStream**
interface and delegate query to the appropriate memory bank regarding of the provided address. Since we will
have several memory bank types, we will define a generic high level **IMemoryBank** interface, which extends
the previously defined **IMemoryStream** :

```java
public interface IMemoryBank extends IMemoryStream {

    int getSize();

    int getOffset();

}
```

The idea here is quite simple, a memory bank is defined by a _size_ (expressed in number of byte), and an
_offset_ which correspond to the starting address this bank is reachable from. The address covered by such
object will be then _[size, size + offset]_. Finally our address bus will be then a simple class that implements
the **IMemoryStream** interface, and is connected to several **IMemoryBank** instance.

## Testing

## Implementation

