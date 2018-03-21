# Chapter 2 : Memory

TODO : Introductive text.

## TODO : Title for memory theory.

### Address space

Before talking of address space, let's review the fundamental about how memory is managed by
a CPU or any processing unit that which to access to a memory. While investingating over
documentation, lot of concept emerge and don't be affraid to admit it, you have sometime no
clue about what is it about. Opcode, Address bus, instruction set, ALU, ... so many knowledges
that at some point in your learning path you probably have seen, but totally forgot since you
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
object will be then _[offset, offset + size]_. Finally our address bus will be then a simple class that implements
the **IMemoryStream** interface, and is connected to several **IMemoryBank** instance.

## Testing

Before start writing some implementations, we can start by writing some tests, and to be more specifc, test interfaces.
As we go with _JUnit5_ test interface is now supported and can be used to design our tests more efficently. Indeed,
we only have interfaces to work on, and since we will have several implemetations for those interfaces, having generic
test is a good start to validate our choices and having a strong validation procedure while writing implementation code.

How testing memory ? The idea is to assume a particular memory state, enough simple to write test for it, but still enough
complex to cover as much usage case as we could. Two cases that we can easily cover are :

- Address out of range
- Reading expected value

We will assume for our tests that we work on a memory which has following properties :

- [4, 6] Address space
- $4 = 10101010
- $5 = 11110000
- $6 = 00000000

The first two addresses aims to be used for memory reading operation testing, and the last one for writing.
We will start by the higher abstraction level available, which is the **IMemoryStream** interface here :

```java
public interface IMemoryStreamTest {

    static final int TEST_SIZE = 3;
    static final int TEST_OFFSET = 4;

    IMemoryStream getTestMemoryStream();
    
    default void performStreamTest(final Consumer<IMemoryStream> test) {
		final IMemoryStream stream = getTestMemoryStream();
		assertNotNull(stream);
		test.accept(stream);
	}

}
```

We define some scenario related constants, as well as a factory method that should be use to retrieve a testing instance.
Finally a shortcut method for performing a test defined as an **IMemoryStream** consumer. Now we can write two tests that
will respectively ensure that uncovered address access will throw an **IllegalAccessException**, and value located at _$4_
and _$5_ are the one expected.

```java
@Test
default void testUnallowedReading() {
    performStreamTest(stream -> {
        assertThrows(IllegalAccessException.class, () -> stream.readByte(TEST_OFFSET - 1));
        assertThrows(IllegalAccessException.class, () -> stream.readByte(TEST_OFFSET + TEST_SIZE));
    });
}

@Test
default void testAllowedReading() {
    performStreamTest(stream -> {
        try {
            assertEquals(85, stream.readByte(TEST_OFFSET));
            assertEquals(15, stream.readByte(TEST_OFFSET + 1));
        }
        catch (final IllegalAccessException e) {
            fail(e);
        }			
    });
}
```

TODO : Note on 85 and 15 value computation.

And that's it ! Such test interface can be now reused for any test that cover a class which implements the **IMemoryStream**
interface. This testing strategy will be used through all this project. We can move to a deeper level, which is the
**IMemoryBank** interface. Since **IMemoryBank** is an **IMemoryStream**, we can write a test interface for it which also
extends the test interface associated to **IMemoryStream** as following :

```java
public interface IMemoryBankTest extends IMemoryStreamTest {

    IMemoryBank getTestMemoryBank();

    @Override
    default IMemoryStream getTestMemoryStream() {
        return getTestMemoryBank();
    }

    default void performBankTest(final Consumer<IMemoryBank> test) {
        final IMemoryBank bank = getTestMemoryBank();
        assertNotNull(bank);
        test.accept(bank);
    }

}
```

As for the previous test interface, we have a factory method, as well as a shortcut method for performing
a memory bank related test. The only difference here, is that we implements the parent abstract method
**IMemoryStreamTest#getTestMemoryStream()** which delegate the call to the bank related factory method.

Here we will only write two simple tests, which cover expected properties value :

```java
@Test
default void testBankSize() {
    performBankTest(bank -> assertEquals(TEST_SIZE, bank.getSize()));
}

@Test
default void testBankOffset() {
    performBankTest(bank -> assertEquals(TEST_OFFSET, bank.getOffset()));
}
```

We are done for the testing part, at least the generic part of it, and we can now move on writing actual
code that will implement our interfaces and more effectivly, implement our emulator memory. keep in mind
that some other concrete test class will be written as long as we write implementation class to validate
them.

## Implementation

Alright, we are now close to have a memory working, and that we now it will work as expected thanks
to written unit tests. But first we need to define exactly our required components. The most obvious
one is the address bus, but we will handle it at the end, as it aims to represent the central access
point and then has dependencies with every other memory related components. The other components are
simply memory banks, which can be divided in two categories :

- Concrete memory bank implementation which can vary due to the used data storage representation.
- Strategy based memory bank implementation, like read only, or switchable.

### Base implementation.

First of all, we can write an **abstract** base class for generic behavior : no need
to duplicate _size_ and _offset_ managment for each implementation we will write :

```java
public abstract class AbstractMemoryBank implements IMemoryBank {

	private final int size;
	private final int offset;

	public AbstractMemoryBank(final int size, final int offset) {
		this.size = size;
		this.offset = offset;
	}

	protected final void verifyAddress(final int address) throws IllegalAccessException {
		if (address < getOffset() || address >= (getOffset() + getSize())) {
			throw new IllegalAccessException();
		}
	}

	@Override public final int getSize() { return size; }
	@Override public final int getOffset() { return offset; }

}
```

Nothing magic here, the only noticeable point is the **protected** method **verifyAddress(int)**
which ensures that a given address is covered by this bank. This will avoid code duplication
over addressing control since we will perform such operation all the time we write an access
method.

### Concrete implementation

A concrete implementation relies on a specific datastructure, for storing and indexing associated
memory block. The most naïve one, and probably the most efficient, is to use a single **byte** array.
Some other based on **java.lang.BitSet** for example could be used, efficienty can be benchmarked
so we can choose the best fit. We can also have _read-oriented_ or _write-oriented_ depending on
assumed memory usage. We will only go for the array based here, but more implementations can be found
on the project repository.

```java
public final class ArrayMemoryBank extends AbstractMemoryBank {

    private final byte[] data;

    public ArrayMemoryBank(final int size, final int offset) {
        super(size, offset);
		this.data = new byte[size];
	}

	@Override
	public byte readByte(final int address) throws IllegalAccessException {
		verifyAddress(address);
		return data[address - getOffset()];
	}

	@Override
	public void writeByte(final byte value, final int address) throws IllegalAccessException {
		verifyAddress(address);
		data[address - getOffset()] = value;
	}
}
```

### Strategy based implementation
