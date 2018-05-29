# Chapter 4 : Instruction set

It is time to start the hardest part or at least the one which needs 

## Design

The choice made here was to isolate each subset of instruction by grouping them into collection of named singletons.
In order to achive this, we used *Java enumeration* mecanisms to provide unique instances for each instruction
through generic purpose interfaces. Here is the list of enumeration used and available in package
``fr.faylixe.yage.cpu.instruction.set`` :

| Set                       | Description                    |
| ------------------------- | ------------------------------ |
| ByteLoadInstructionSet    | 8-bit load instructions        |
| ShortLoadInstructionSet   | 16-bit load instructions       |
| ByteALUInstructionSet     | 8-bit arithmetic operations    |
| ShortALUInstructionSet    | 16-bit arithmetic operations   |
| BitInstructionSet         | Bit manipulation               |
| MiscInstructionSet        | Misc instruction               |
| RotateShiftInstructionSet | Bit rotation / shift operation |
| CallInstructionSet        | Call related instruction       |
| JumpInstructionSet        | Jump related instruction       |
| RestartInstructionSet     | Hardware restart instruction   |
| ReturnInstructionSet      | Return related instruction     |

Table: Instruction subset associated enumeration class 

> TODO : Consider being more specific with description.

Aggregating instruction into enumeration offer several advantages such as :

- *Code lisibility* : you can separate easily instruction code and semantic.
- *Debugging* : An named instruction entity in a JVM point of view allow to track it faster.
- *Testing* : You can test each instance without effort, as long as you have an *execution context*.

We won't dive into each instruction set since GameBoy instruction set offer 512 instructions,
but we will takes few representatives from each subset to understand the logic applied.

To design such enumeration efficently we assumed the following idea : from an emulator point of view,
the associated code of a given instruction should be able to access to each aspect of the hardware,
namely :

- Instruction stream
- Memory through address bus
- CPU associated registers

To avoid having method signature with lot of parameters, we need an interface that acts
as a single entry point for all thoses components :

```java
public interface IExecutionContext extends IRegisterProvider, IInstructionStream, IMemoryStream {}
```

As you can see, an *execution context* only reuse previoulsy defined interfaces and does not
need anymore effort. From this, we can define an instruction in an oriented object manner using
following specification :

- An instruction is mapped by an unique opcode.
- An instruction is regulated by a fixed machine cycle number.
- An instruction is defined as an executable code.

We could go for a unique interface that groups those enumerated aspect, but we will split this contract in
two parts : one that handles the instruction execution code, and another one on top of it that will offer
the opcode and machine cycle :

```java
@FunctionalInterface
public interface IExecutableInstruction {
  void execute(IExecutionContext context) throws IllegalAccessException;
}

public interface IInstruction extends IExecutableInstruction {
	short getOpcode();
	byte getCycle();
}
```

The motivation behinds this separation of concern is the ability to define an instruction execution code
as a lambda expression, or method reference since the first interface is designed as a functional one.

### 8-bit load instruction set

As a detailed example on how implementing an instruction set using *enumeration*, we will use the easiest one which is the
8-bit load. Indeed, operation in this set only consist in copying value from and to either register or memory.

The first step is to defined our empty instruction set using *decoration* pattern with the ``IInstruction`` interface :

```java
public enum ByteLoadInstructionSet implements IInstruction {

	;
	private short opcode;
	private byte cycle;
	private IExecutableInstruction executable;
	
	private ByteLoadInstructionSet(int opcode, int cycle, IExecutableInstruction executable) {
		this.opcode = (short) opcode;
		this.cycle = (byte) cycle;
		this.executable = executable;
	}

	@Override
	public void execute(IExecutionContext context) throws IllegalAccessException {
		executable.execute(context);
	}

	@Override public short getOpcode() { return opcode; }
	@Override public byte getCycle() { return cycle; }

}
```

From this, the only effort is organisation ! Keep track of which instruction has been already implemented,
tested, documented. In *YAGE*, order from the Gameboy CPU Manual has been picked, as for the enumeration
grouping as well. Let's start with the first instruction : ``LD nn,n``, which is defined as pushing the next
8-bit immediate value into a target 8-bit register. The instruction opcodes are described as following :

| Instruction | Parameters | Opcode | Cycles |
| ----------- | ---------- | ------ | ------ |
| LD          | B, n       | 06     | 8      |
| LD          | C, n       | 0E     | 8      |
| LD          | D, n       | 16     | 8      |
| LD          | E, n       | 1E     | 8      |
| LD          | H, n       | 26     | 8      |
| LD          | L, n       | 2E     | 8      |

Table: LD nn,n

Picking the first one, adding it to our enumeration is as simple as following :

```java
LD_B_NN(0x06, 8, context -> context.getRegister(B).set(context.nextByte())),
```

> Note that in order to use register name directly you will have to perform a static import
> of the associated enumeration from ``IRegisterProvider`` interface.

Continuing from that first version we can go for all the others :

```java
LD_C_NN(0x0E, 8, context -> context.getRegister(C).set(context.nextByte())),
LD_D_NN(0x16, 8, context -> context.getRegister(D).set(context.nextByte())),
LD_E_NN(0x1E, 8, context -> context.getRegister(E).set(context.nextByte())),
LD_H_NN(0x26, 8, context -> context.getRegister(H).set(context.nextByte())),
LD_L_NN(0x2E, 8, context -> context.getRegister(L).set(context.nextByte())),
```

Congratulation ! You have implemented already six instructions of the Gameboy CPU. Although one flaw of this design
is the code duplication, but i can be improved too : instead of writing code for each instruction variation
you can choose to use a static factory method that will create the required ``IExecutableInstruction`` for a given
set of parameter :

```java
static IExecutableInstruction copyNextValue(Register destination) {
	return context -> context.getRegister(destination).set(context.nextByte());
}
```

Using this factory we can rewrite our instructions as following :

```java
LD_B_NN(0x06, 8, copyNextValue(B)),
LD_C_NN(0x0E, 8, copyNextValue(C)),
LD_D_NN(0x16, 8, copyNextValue(D)),
LD_E_NN(0x1E, 8, copyNextValue(E)),
LD_H_NN(0x26, 8, copyNextValue(H)),
LD_L_NN(0x2E, 8, copyNextValue(L)),
```

## Testing

As for previous system, we will define an interface that defines behavior of
tests for a given instruction set. In order to ensure that an instruction execution
performed well, we will check state of other components that are deeply connected through
the *execution context* : memory, register and instruction stream.

```java
@TestInstance(Lifecycle.PER_CLASS)
public interface IInstructionSetTest extends
	IInstructionStreamTest,
	IMemoryStreamTest,
	IRegisterProviderTest {

	@Override
	default IRegisterProvider getTestRegisterProvider() {
		return IRegisterProviderTest.createMockRegisterProvider();
	}

	@Override
	default IInstructionStream getTestInstructionStream() {
		return IInstructionStreamTest.createMockInstructionStream();
	}

	@Override
	default IMemoryStream getTestMemoryStream() {
		int size = (int) pow(2, 16);
		AddressBus bus = new AddressBus(size);
		bus.connect(IMemoryBankTest.createMemoryBankMock(TEST_OFFSET, 0));
		bus.connect(IMemoryBankTest.createMemoryBankMock());
		int offset = TEST_OFFSET + TEST_SIZE;
		bus.connect(IMemoryBankTest.createMemoryBankMock(size - offset, offset));
		return bus;
	}

}
```

You can note that instruction stream and register testing contract can be used as in, but
the memory part should be updated in order to offer the whole address space, where each memory
cell hold 0 value, and address from the ``IMemoryStreamTest`` are still preserved. An additional
step is required due to the fact that all those inherited test component should
be linked through ``IExecutionContext`` interface. We will then use mockito to bind parent
interface mocks to the target contract using following static methods :

```java
private static void bindMemoryStream(IExecutionContext context, IMemoryStream stream) {
	try {
		when(context.readByte(anyInt())).then(toAnswer(stream::readByte));
		when(context.readBytes(anyInt(), anyInt())).then(toAnswer(stream::readBytes));
		doAnswer(toAnswer(stream::writeByte)).when(context).writeByte(anyByte(), anyInt());
		doAnswer(toAnswer(stream::writeBytes)).when(context).writeBytes(any(), anyInt());
	}
	catch (final IllegalAccessException e) {
		fail(e);
	}
}

private static void bindRegisterProvider(IExecutionContext context, IRegisterProvider provider) {
	when(context.getRegister(any())).then(toAnswer(provider::getRegister));
	when(context.getFlagsRegister()).then(toAnswer(provider::getFlagsRegister));
	when(context.getExtendedRegister(any())).then(toAnswer(provider::getExtendedRegister));
}

private static void bindInstructionStream(IExecutionContext context, IInstructionStream stream) {
	try {
		when(context.nextByte()).then(toAnswer(stream::nextByte));
		when(context.nextShort()).then(toAnswer(stream::nextShort));
	}
	catch (final IllegalAccessException e) {
		fail(e);
	}
}
```

Using all thoses factory, we can finally define a generic instruction test method
that retrieves all testing instance, bind them to a mock ``IExecutionContext`` and
transmit it to a user provided test. Basic attribute such as opcode and machine cycle
are automatically tested to enforce mistyping detection :

```java
default void performInstructionTest(
	int expectedOpcode,
	int expectedCycle,
	IInstruction instruction,
	ThrowingConsumer<IExecutionContext> test) {
		assertNotNull(instruction);
		assertEquals(expectedOpcode, instruction.getOpcode());
		assertEquals(expectedCycle, instruction.getCycle());
		IExecutionContext context = mock(IExecutionContext.class);
		bindMemoryStream(context, getTestMemoryStream());
		bindInstructionStream(context, getTestInstructionStream());
		bindRegisterProvider(context, getTestRegisterProvider());
		try {
			instruction.execute(context);
			test.accept(context);
		}
		catch (final Exception e) {
			fail(e);
		}
	}
```

```java
@TestFactory
public Collection<DynamicTest> testLoadFromImmediate() {
	return asList(
		dynamicTest("LD B, n", () -> performInstructionTest(0x06, 8, LD_B_N, createRegistersTest(B, (byte) 42))),
		dynamicTest("LD C, n", () -> performInstructionTest(0x0E, 8, LD_C_N, createRegistersTest(C, (byte) 42))),
		dynamicTest("LD D, n", () -> performInstructionTest(0x16, 8, LD_D_N, createRegistersTest(D, (byte) 42))),
		dynamicTest("LD E, n", () -> performInstructionTest(0x1E, 8, LD_E_N, createRegistersTest(E, (byte) 42))),
		dynamicTest("LD H, n", () -> performInstructionTest(0x26, 8, LD_H_N, createRegistersTest(H, (byte) 42))),
		dynamicTest("LD L, n", () -> performInstructionTest(0x2E, 8, LD_L_N, createRegistersTest(L, (byte) 42)))
	);
}
