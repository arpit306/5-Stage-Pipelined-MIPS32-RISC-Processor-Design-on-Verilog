# Design of 5 Stage Pipelined MIPS32 RISC Processor  
This repository contains the details and the code for the MIPS32 ISA based RISC Processor, which is implemented in 5 stage pipelined configuration.  
## Table of contents  
▫️ [MIPS32](https://github.com/arpit306/5-Stage-Pipelined-MIPS32-RISC-Processor-Design-on-Verilog#%EF%B8%8F-mips32)  
▫️ [Addressing Modes](https://github.com/arpit306/5-Stage-Pipelined-MIPS32-RISC-Processor-Design-on-Verilog#%EF%B8%8F-addressing-modes)  
▫️ [Instructions considered](https://github.com/arpit306/5-Stage-Pipelined-MIPS32-RISC-Processor-Design-on-Verilog#%EF%B8%8F-instructions-considered)  
▫️ [Instruction Encoding](https://github.com/arpit306/5-Stage-Pipelined-MIPS32-RISC-Processor-Design-on-Verilog#%EF%B8%8F-instruction-encoding)  
▫️ [Stages of Execution](https://github.com/arpit306/5-Stage-Pipelined-MIPS32-RISC-Processor-Design-on-Verilog#%EF%B8%8F-stages-of-execution)  
▫️ [Non Pipelined DataPath](https://github.com/arpit306/5-Stage-Pipelined-MIPS32-RISC-Processor-Design-on-Verilog#%EF%B8%8F-non-pipelined-datapath)  
▫️ [Pipelined DataPath](https://github.com/arpit306/5-Stage-Pipelined-MIPS32-RISC-Processor-Design-on-Verilog#%EF%B8%8F-pipelined-datapath)  
▫️ [Verilog Design Code](https://github.com/arpit306/5-Stage-Pipelined-MIPS32-RISC-Processor-Design-on-Verilog/blob/main/mips32_design.v)  
▫️ [Example Program Testbench Code](https://github.com/arpit306/5-Stage-Pipelined-MIPS32-RISC-Processor-Design-on-Verilog/blob/main/testbench.v)  
▫️ [EDAplayground Link](https://edaplayground.com/x/t8Vx)  
▫️ [Known issues and problems](https://github.com/arpit306/5-Stage-Pipelined-MIPS32-RISC-Processor-Design-on-Verilog#%EF%B8%8F-known-problems-and-issues)  
▫️ [References](https://github.com/arpit306/5-Stage-Pipelined-MIPS32-RISC-Processor-Design-on-Verilog#%EF%B8%8F-references)  
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
## ▫️ Instructions Considered  
Not all instructions of MIPS32 are considered in this design, for implementation sake only a few instructions are considered, mentioned below:  
- Load and Store Instructions  
```
LW R2,124(R8) // R2 = Mem[R8+124]  
SW R5,-10(R25) // Mem[R25-10] = R5  
```
- Arithmetic and Logic Instructions (only register operands)  
```
ADD R1,R2,R3 // R1 = R2 + R3  
ADD R1,R2,R0 // R1 = R2 + 0  
SUB R12,R10,R8 // R12 = R10 – R8  
AND R20,R1,R5 // R20 = R1 & R5  
OR R11,R5,R6 // R11 = R5 | R6  
MUL R5,R6,R7 // R5 = R6 * R7  
SLT R5,R11,R12 // If R11 < R12, R5=1; else R5=0 
```
- Arithmetic and Logic Instructions (immediate operand)  
```
ADDI R1,R2,25 // R1 = R2 + 25  
SUBI R5,R1,150 // R5 = R1 – 150  
SLTI R2,R10,10 // If R10<10, R2=1; else R2=0 
```
- Branch Instructions  
```
BEQZ R1,Loop // Branch to Loop if R1=0  
BNEQZ R5,Label // Branch to Label if R5!=0  
```
- Jump Instruction  
```
J Loop // Branch to Loop unconditionally  
```
- Miscellaneous Instructioon  
```
HLT // Halt execution 
```
## ▫️ Instruction Encoding  
![ISR](https://user-images.githubusercontent.com/68592620/231771092-0c93aeb3-6b01-478f-a363-ecadb1ec578a.png)  
- shamt : shift amount, funct : opcode extension for additional functions.
- Some instructions require two register operands rs & rt as input, while some require only rs. 
- This requirement is only identified only after the instruction is decoded. 
- While decoding is going on, we can prefetch the registers in parallel, which may or may not be used later. 
- Similarly, the 16-bit and 26-bit immediate data are retrieved and signextended to 32-bits in case they are required later.  
## ▫️ Stages of Execution  
The instruction execution cycle contains the following 5 stages in order:  
1. IF : Instruction Fetch  
2. ID : Instruction Decode / Register Fetch  
3. EX : Execution / Effective Address Calculation  
4. MEM : Memory Access / Branch Completion  
5. WB : Register Write-back  
- micro operations not shown here.
## ▫️ Non Pipelined DataPath  
![nonpipelined](https://user-images.githubusercontent.com/68592620/231771101-f7ea7e00-5c8c-4b6d-ae0c-a0419066e7ad.png)  
## ▫️ Pipelined DataPath  
![pipelined](https://user-images.githubusercontent.com/68592620/231771102-12c05fa9-6e74-4835-abc6-1bd9b20e8453.png)  
## ▫️ Verilog Design Code  
``` 
  module pipe_MIPS32 (clk1, clk2);
  input clk1, clk2; // Two-phase clock
  reg [31:0] PC, IF_ID_IR, IF_ID_NPC;
  reg [31:0] ID_EX_IR, ID_EX_NPC, ID_EX_A, ID_EX_B, ID_EX_Imm;
  reg [2:0] ID_EX_type, EX_MEM_type, MEM_WB_type;
  reg [31:0] EX_MEM_IR, EX_MEM_ALUOut, EX_MEM_B;
  reg EX_MEM_cond;
  reg [31:0] MEM_WB_IR, MEM_WB_ALUOut, MEM_WB_LMD;
  reg [31:0] Reg [0:31]; // Register bank (32 x 32)
  reg [31:0] Mem [0:1023]; // 1024 x 32 memory
  parameter ADD=6'b000000, SUB=6'b000001, AND=6'b000010, OR=6'b000011,
            SLT=6'b000100, MUL=6'b000101, HLT=6'b111111, LW=6'b001000,
            SW=6'b001001, ADDI=6'b001010, SUBI=6'b001011,SLTI=6'b001100,
            BNEQZ=6'b001101, BEQZ=6'b001110;
  parameter RR_ALU=3'b000, RM_ALU=3'b001, LOAD=3'b010, STORE=3'b011,
            BRANCH=3'b100, HALT=3'b101;
  reg HALTED;
  // Set after HLT instruction is completed (in WB stage)
  reg TAKEN_BRANCH;
  // Required to disable instructions after branch
  
  
  always @(posedge clk1) // IF Stage
    if (HALTED == 0)
    begin
      if (((EX_MEM_IR[31:26] == BEQZ) && (EX_MEM_cond == 1)) ||
          ((EX_MEM_IR[31:26] == BNEQZ) && (EX_MEM_cond == 0)))
      begin
        IF_ID_IR <= #2 Mem[EX_MEM_ALUOut];
        TAKEN_BRANCH <= #2 1'b1;
        IF_ID_NPC <= #2 EX_MEM_ALUOut + 1;
        PC <= #2 EX_MEM_ALUOut + 1;
      end
      else
      begin
        IF_ID_IR <= #2 Mem[PC];
        IF_ID_NPC <= #2 PC + 1;
        PC <= #2 PC + 1;
      end
    end
  
  always @(posedge clk2) // ID Stage
    if (HALTED == 0)
    begin
      if (IF_ID_IR[25:21] == 5'b00000)
        ID_EX_A <= 0;
      else
        ID_EX_A <= #2 Reg[IF_ID_IR[25:21]]; // "rs"
      if (IF_ID_IR[20:16] == 5'b00000)
        ID_EX_B <= 0;
      else
        ID_EX_B <= #2 Reg[IF_ID_IR[20:16]]; // "rt"
      ID_EX_NPC <= #2 IF_ID_NPC;
      ID_EX_IR <= #2 IF_ID_IR;
      ID_EX_Imm <= #2 {{16{IF_ID_IR[15]}}, {IF_ID_IR[15:0]}};
      case (IF_ID_IR[31:26])
        ADD,SUB,AND,OR,SLT,MUL:
          ID_EX_type <= #2 RR_ALU;
        ADDI,SUBI,SLTI:
          ID_EX_type <= #2 RM_ALU;
        LW:
          ID_EX_type <= #2 LOAD;
        SW:
          ID_EX_type <= #2 STORE;
        BNEQZ,BEQZ:
          ID_EX_type <= #2 BRANCH;
        HLT:
          ID_EX_type <= #2 HALT;
        default:
          ID_EX_type <= #2 HALT;
        // Invalid opcode
      endcase
    end
  
  always @(posedge clk1) // EX Stage
    if (HALTED == 0)
    begin
      EX_MEM_type <= #2 ID_EX_type;
      EX_MEM_IR <= #2 ID_EX_IR;
      TAKEN_BRANCH <= #2 0;
      case (ID_EX_type)
        RR_ALU:
        begin
          case (ID_EX_IR[31:26]) // "opcode"
            ADD:
              EX_MEM_ALUOut <= #2 ID_EX_A + ID_EX_B;
            SUB:
              EX_MEM_ALUOut <= #2 ID_EX_A - ID_EX_B;
            AND:
              EX_MEM_ALUOut <= #2 ID_EX_A & ID_EX_B;
            OR:
              EX_MEM_ALUOut <= #2 ID_EX_A | ID_EX_B;
            SLT:
              EX_MEM_ALUOut <= #2 ID_EX_A < ID_EX_B;
            MUL:
              EX_MEM_ALUOut <= #2 ID_EX_A * ID_EX_B;
            default:
              EX_MEM_ALUOut <= #2 32'hxxxxxxxx;
          endcase
        end
        RM_ALU:
        begin
          case (ID_EX_IR[31:26]) // "opcode"
            ADDI:
              EX_MEM_ALUOut <= #2 ID_EX_A + ID_EX_Imm;
            SUBI:
              EX_MEM_ALUOut <= #2 ID_EX_A - ID_EX_Imm;
            SLTI:
              EX_MEM_ALUOut <= #2 ID_EX_A < ID_EX_Imm;
            default:
              EX_MEM_ALUOut <= #2 32'hxxxxxxxx;
          endcase
        end
        LOAD, STORE:
        begin
          EX_MEM_ALUOut <= #2 ID_EX_A + ID_EX_Imm;
          EX_MEM_B <= #2 ID_EX_B;
        end
        BRANCH:
        begin
          EX_MEM_ALUOut <= #2 ID_EX_NPC + ID_EX_Imm;
          EX_MEM_cond <= #2 (ID_EX_A == 0);
        end
      endcase
    end
  
  
  always @(posedge clk2) // MEM Stage
    if (HALTED == 0)
    begin
      MEM_WB_type <= EX_MEM_type;
      MEM_WB_IR <= #2 EX_MEM_IR;
      case (EX_MEM_type)
        RR_ALU, RM_ALU:
          MEM_WB_ALUOut <= #2 EX_MEM_ALUOut;
        LOAD:
          MEM_WB_LMD <= #2 Mem[EX_MEM_ALUOut];
        STORE:
          if (TAKEN_BRANCH == 0) // Disable write
            Mem[EX_MEM_ALUOut] <= #2 EX_MEM_B;
      endcase
    end
  
  
  always @(posedge clk1) // WB Stage
  begin
    if (TAKEN_BRANCH == 0) // Disable write if branch taken
    case (MEM_WB_type)
      RR_ALU:
        Reg[MEM_WB_IR[15:11]] <= #2 MEM_WB_ALUOut; // "rd"
      RM_ALU:
        Reg[MEM_WB_IR[20:16]] <= #2 MEM_WB_ALUOut; // "rt"
      LOAD:
        Reg[MEM_WB_IR[20:16]] <= #2 MEM_WB_LMD; // "rt"
      HALT:
        HALTED <= #2 1'b1;
    endcase
  end
endmodule
  ```  
## ▫️ Example Program Testbench Code  
Steps:  
1. Initialize register R1 with 10.  
2. Initialize register R2 with 20.  
3. Initialize register R3 with 25.  
4. Add the three numbers and store the sum in R5. 
 
Instructions :  
| Assembly Instruction  | Machine Code | Hexcode |  
| ------------- | ------------- | ------------- |  
| ADDI R1,R0,10  | 001010 00000 00001 0000000000001010  | 2801000a  |  
| ADDI R2,R0,20 | 001010 00000 00010 0000000000010100  | 28020014  |  
| ADDI R3,R0,25 | 001010 00000 00011 0000000000011001  | 28030019  |  
| OR R7,R7,R7 (dummy)| 001010 00000 00011 0000000000011001  | 0ce77800  |  
| OR R7,R7,R7 (dummy)| 001010 00000 00011 0000000000011001  | 0ce77800  |  
| ADD R4,R1,R2 | 000000 00001 00010 00100 00000 000000  | 00222000  |  
| OR R7,R7,R7 (dummy)| 001010 00000 00011 0000000000011001  | 0ce77800 |  
| ADD R5,R4,R3 | 000000 00100 00011 00101 00000 000000  | 00832800  |  
| HLT | 111111 00000 00000 00000 00000 000000  | fc000000  |  

Testbench Code :  
``` 
  module test_mips32;
  reg clk1, clk2;
  integer k;
  pipe_MIPS32 mips (clk1, clk2);
  initial
  begin
    clk1 = 0;
    clk2 = 0;
    repeat (20) // Generating two-phase clock
    begin
      #5 clk1 = 1;
      #5 clk1 = 0;
      #5 clk2 = 1;
      #5 clk2 = 0;
    end
  end
  initial
  begin
    for (k=0; k<31; k++)
      mips.Reg[k] = k;
    mips.Mem[0] = 32'h2801000a; // ADDI R1,R0,10
    mips.Mem[1] = 32'h28020014; // ADDI R2,R0,20
    mips.Mem[2] = 32'h28030019; // ADDI R3,R0,25
    mips.Mem[3] = 32'h0ce77800; // OR R7,R7,R7 -- dummy instr.
    mips.Mem[4] = 32'h0ce77800; // OR R7,R7,R7 -- dummy instr.
    mips.Mem[5] = 32'h00222000; // ADD R4,R1,R2
    mips.Mem[6] = 32'h0ce77800; // OR R7,R7,R7 -- dummy instr.
    mips.Mem[7] = 32'h00832800; // ADD R5,R4,R3
    mips.Mem[8] = 32'hfc000000; // HLT
    mips.HALTED = 0;
    mips.PC = 0;
    mips.TAKEN_BRANCH = 0;
    #280
     for (k=0; k<6; k++)
       $display ("R%1d - %2d", k, mips.Reg[k]);
  end
  initial
  begin
    $dumpfile ("mips.vcd");
    $dumpvars (0, test_mips32);
    #300 $finish;
  end
endmodule
  ```  
Waveform :  
![waveform](https://user-images.githubusercontent.com/68592620/231780893-8d26f3ff-9b60-44a5-93a7-c40f17219e6e.png)  

Console output :  
``` R0 -  0
R1 - 10
R2 - 20
R3 - 25
R4 - 30
R5 - 55  
```  
## ▫️ EDAplayground Link  
[https://edaplayground.com/x/t8Vx](https://edaplayground.com/x/t8Vx )  
## ▫️ Known problems and issues  
Following pipelining hazards are present in the given design :  
- Structural Hazards due to shared hardware.  
- Data Hazards due to instruction data dependency.  
- Control hazards due to branch instructions.  
## ▫️ References  
[NPTEL \& IIT KGP 'Hardware Modeling using Verilog'- Prof. Indranil Sengupta](https://nptel.ac.in/courses/106105165)
