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
