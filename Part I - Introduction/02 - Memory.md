# Chapter 2 : Memory

TODO : Introductive text.

## TODO : Title for memory theory.

### Address space

Before talking about address space, let's review the fundamental about how memory is managed by
a CPU or any processing unit that whishes to access a memory. Whilst investingating 
documentation, lots of concept emerges and (don't be affraid to admit it) you sometime have no
clue about what it is about. Opcode, Address bus, instruction set, ALU, ... So many terminlogies
that at some point in your learning path you'll probably have seen, but totally forgot about because you
never used them in a real project, or in your daily job.

Addressing is one of them, and more generally, how memory is designed in a low level / hardware
point of view. Memory, or memories as you can have several in your system, is connected
to the CPU (and all other processing units) through an address bus and data bus.

When a unit needs to read the memory, it puts the requested memory address on the address bus. The
associated data is then put on the data bus that the unit can read. When a unit needs to write to
memory, it puts the target memory address on the address bus and the target data on the data bus.
Nothing more, nothing less.

The address space is the range of addresses that can be reached through your address bus. In case of
the gameboy, the address bus is 16-bit wide. Thus your memory address varies between 0 and 2<sup>16</sup> - 1, 
so 65536 addresses available.

### Memory map

As we said previously, you can and you will have several memory in your system. Global working RAM,
ROM, video dedicated RAM, and so on, but the address space is unique, and the way a subset of the
address space is allocated to a specific memory instance is what we call the _memory map_.

The _memory map_ is a specification that splits the address space in subsets that are dedicated
to a particular block of memory. Following table describes the _memory map_ associated to the
gameboy address bus, but keep in mind that you can easily have a more detailled memory map with slot
description and/or map variation depending on console model.

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

Please note the used address notation, as it will be the same throughout all the book and most likely on
documentation you will find over the web. An address is prefixed by the dollar sign, and consists
of 4 digits, using hexadecimal notation, which represents the effective 16-bit address. Remember, 16-bit,
where 8-bit define a byte, which can be written as two hexadecimal digits. I recommand having this _memory map_ 
close to you at all times using a post it, a printed sheet, or a browser tab, it is your call.

### Memory banking

On some _memory map_ items, you can notice that some target are labelled with "bank" followed by a
numeric identifier : this refers to another concept that is _memory banking_.

TODO : Switchable memory bank.

## Design

Alright, now we are a little bit more confortable in how memory works, we can start doing some system
design. Since we are using **Java**, we will think in an oriented object manner,
which is good as we can see a gameboy as a composition of several objects and here is what we need to
represent our overall memory :

- An **AddressBus**
- Various memory banks

But before diving into it, we will define behaviors we need from our memory :

- Read a byte at a given address
- Write a byte at a given address

This is quite straightforward, we can write an interface for it :

```java
public interface IMemoryStream {

    byte readByte(int address) throws IllegalAccessException;

    void writeByte(byte value, int address) throws IllegalAccessException;

}
```

Despite how simple the interface is, we already face lots of issues here. First of all the storage aspect,
each memory cell is represented by a **byte** value, which is 8-bit long in _Java_. Since a gameboy address
reaches an 8-bit block of data this is perfect, no memory waste, but all primitive types in _Java_ are signed.
We should then keep this fact in mind when we will implement artihmetic operations later. The second main
issue is the address storage type : we went for an **int** here which is 32-bit long. This means that we will
use twice the amount of memory required to store an address since the gameboy only has a 16-bit wide address space.
Why not going for the **short** type which is *16-bit* long ? Well as for the storage aspect, **short**
values are signed, which implies that values can go between -32768 and 32767. We want an [0, 65536]
address space. In order to ensure a given address is valid regardless of the address space, we use
a signed **int** which can cover the required range. Another option is to effectively use a **short**
and build the real address on the fly by computing the unsigned value from bits, but since we aim to use
addresses for memory indexing in a time efficient datastructure, this choice wasn't retained.

We now have made some choices, in how we reach the memory. We will continue then by modeling how we will
organize objects to represents it. As said earlier, we want to represent an address bus, which will be
the central point for accessing memory. Our address bus will receive I/O operation through the **IMemoryStream**
interface and delegate query to the appropriate memory bank. Since we will have several memory bank types, 
we will define a generic high level **IMemoryBank** interface, which extends the previously defined **IMemoryStream** :

```java
public interface IMemoryBank extends IMemoryStream {

    int getSize();

    int getOffset();

}
```

The idea here is quite simple, a memory bank is defined by a _size_ (expressed in number of byte), and an
_offset_ which correspond to the starting address this bank is reachable from. The address covered by such
object will then be _[offset, offset + size]_. Finally our address bus will then be a simple class that implements
the **IMemoryStream** interface and is connected to several **IMemoryBank** instances.

## Testing

Before we start writing some implementations, we can write some tests, or to be more specific, test interfaces.
As we'll use _JUnit5_, test interfaces are now supported and can be used to design our tests more efficiently. Indeed,
we only have interfaces to work on and since we will have several implemetations for those interfaces, having a generic
test is a good start to validate our choices and having a strong validation procedure while writing implementation code.

How do we test memory ? The idea is to assume a particular memory state, simple enough to write tests for it, but still complex 
enough to cover as much usage case as we could. Two cases that we can easily cover are :

- Address out of range
- Reading expected value

We will assume for our tests that we work on a memory which has following properties :

- [4, 6] Address space
- $4 = 0b10101010
- $5 = 0b11110000
- $6 = 0b00000000

The first two addresses aim to be used for memory reading operation testing, and the last one for writing.
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
will respectively ensure that uncovered address access will throw an **IllegalAccessException** and that the value located at _$4_
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

That's it ! Such test interface can now be reused for any test that covers a class which implements the **IMemoryStream**
interface. This testing strategy will be used throughout all this project. We can now move to a deeper level, which is the

**IMemoryBank** interface. Since **IMemoryBank** is an **IMemoryStream**, we can write a test interface for it which also
extends the test interface associated to **IMemoryStream** as follows :

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

As for the previous test interface, we have a factory method as well as a shortcut method for performing
a memory bank related test. The only difference here is that we implement the parent abstract method
**IMemoryStreamTest#getTestMemoryStream()** which delegates the call to the bank related factory method.

Here we will only write two simple tests which cover expected properties value :

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

Alright, we are now close to have a working memory, and that we now it will work as expected thanks
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

#### ROM (Read Only Memory)

```java
public final class ReadOnlyMemoryBank implements IMemoryBank {

	private final IMemoryBank delegate;

	public ReadOnlyMemoryBank(final IMemoryBank delegate) {
		this.delegate = delegate;
	}

	@Override
	public int getSize() {
		return delegate.getSize();
	}

	@Override
	public int getOffset() {
		return delegate.getOffset();
	}

	@Override
	public void writeByte(final byte value, final int address) throws IllegalAccessException {
		throw new IllegalAccessException("Attemping to write into read only memory block.");
	}

	@Override
	public byte readByte(final int address) throws IllegalAccessException {
		return delegate.readByte(address);
	}

}
```

#### Switchable bank

```java

public final class SwitchableMemoryBank extends AbstractMemoryBank {

	private final IMemoryStream [] banks;
	private final AtomicInteger current;

	private SwitchableMemoryBank(
			final IMemoryBank [] banks,
			final int size,
			final int offset) {
		super(size, offset);
		this.banks = banks;
		this.current = new AtomicInteger();
	}

	public int getBankSize() {
		return banks.length;
	}

	public void switchBank(final int target) {
		if (target < 0 || target >= banks.length) {
			throw new IllegalArgumentException();
		}
		current.set(target);
	}

	@Override
	public byte readByte(final int address) throws IllegalAccessException {
		return banks[current.get()].readByte(address);
	}

	@Override
	public void writeByte(final byte value, final int address) throws IllegalAccessException {
		banks[current.get()].writeByte(value, address);
	}

}
```

```java
public static final class Builder {

    private final int size;
    private final int offset;
    private final List<IMemoryBank> banks;

    private Builder(final int size, final int offset) {
        this.size = size;
        this.offset = offset;
        this.banks = new ArrayList<IMemoryBank>();
	}

    public Builder addMemoryBank(final IMemoryBank bank) {
        if (bank == null) {
            throw new IllegalArgumentException("Null bank");
        }
        if (bank.getSize() != size) {
            throw new IllegalArgumentException("Size not matching");
        }
        if (bank.getOffset() != offset) {
            throw new IllegalArgumentException("Offset not matching");
        }
        banks.add(bank);
        return this;
	}

	public SwitchableMemoryBank build() {
		if (banks.isEmpty()) {
			throw new IllegalStateException("No bank available");
		}
		final IMemoryBank [] switchable = new IMemoryBank[banks.size()];
		banks.toArray(switchable);
		return new SwitchableMemoryBank(switchable, size, offset);
	}

}
```

### Address bus

```java
public final class AddressBus implements IMemoryStream {

	private final NavigableMap<Integer, IMemoryBank> memory;

	public AddressBus() {
		this.memory = new TreeMap<>();
	}

	public void connect(final IMemoryBank memoryBank) {
		if (memoryBank == null) {
			throw new IllegalArgumentException();
		}
		// final int offset = memoryBlock.getOffset();
		// TODO : Check for address conflict.
		memory.put(memoryBank.getOffset(), memoryBank);
	}

	public void disconnect(final IMemoryBank memoryBank) {
		if (memoryBank == null) {
			throw new IllegalArgumentException();
		}
		if (memory.remove(memoryBank.getOffset()) == null) {
			throw new IllegalArgumentException();
		}
	}

	private IMemoryStream getMemoryStream(final int address) throws IllegalAccessException {
		final Integer nearAddressableOffset = memory.floorKey(address);
		if (nearAddressableOffset == null) {
			throw new IllegalAccessException("Unreachable memory address");
		}
		// TODO : Check for more coverage ?
		return memory.get(nearAddressableOffset);
	}

	@Override
	public byte readByte(final int address) throws IllegalAccessException {
		return getMemoryStream(address).readByte(address);
	}

	@Override
	public void writeByte(final byte value, final int address) throws IllegalAccessException {
		getMemoryStream(address).writeByte(value, address);
	}

}
```