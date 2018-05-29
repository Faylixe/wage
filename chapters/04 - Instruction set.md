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
