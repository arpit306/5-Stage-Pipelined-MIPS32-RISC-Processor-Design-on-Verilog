# 5-Stage-Pipelined-MIPS32-RISC-Processor-Design-on-Verilog  
In this repository, I have mentioned the details and code of the MIPS32 ISA based RISC Processor, which is implemented in 5 stage pipelined configuration.  
## Table of contents  
▫️ MIPS32  
▫️ Instructions considered  
▫️ Addressing Modes  
▫️ Instruction Encoding  
▫️ Stages of Execution  
▫️ Non Pipelined DataPath  
▫️ Pipelined DataPath  
▫️ Verilog Design Code  
▫️ Example Program Testbench Code  
▫️ EDAplayground Link  
▫️ Known issues and problems  
▫️ References  
## ▫️ MIPS32  
- 32 x 32 bit GPRs [R0 to R31]  
- R0 hardwired to logic0  
- 32 bit Program Counter (PC)  
- No flag registers (carry, zero, sign..etc)  
- Few Addresing Modes  
- Only Load and Store instructions can access memory  
- We assume memory word size is 32 bits (word addressable)  
## ▫️ Addressing Modes  
| Addressing Mode | Example Instruction |
| -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Register addressing | ADD R1,R2,R3      |
| Immediate addressing | ADDI R1,R2, 200       |
| Base addressing      | LW R5, 150(R7)    |
| PC relative addressing  | BEQZ R3, Label   |
| Pseudo-direct addressing | J Label      |

## ▫️ Instructions considered 
Not all instructions of MIPS32 are considered in this design, for implementation sake only a few instructions are considered, mentioned below:
