---
layout: post
title: Intel x86 Assembly Status Flags
---

Behold, a table showing how general purpose Intel x86 assembly instructions affect the status flags.

Note: My blog theme currently makes tables ugly as shit, so I recommend you look at this in the github repository browser. The markdown table looks right there.

INSTRUCTION | CF | OF | SF | ZF | PF | AF | NOTES
:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:
MOV | u | u | u | u | u | u | u = unaffected
CMOVcc | u | u | u | u | u | u | 
XCHG | u | u | u | u | u | u | 
BSWAP | u | u | u | u | u | u | 
XADD | like ADD | like ADD | like ADD | like ADD | like ADD | like ADD | 
CMPXCHG[8B] | like CMP | like CMP | like CMP | (EAX = op1) ? 1 : 0 | like CMP | like CMP | 
PUSH[A[D]] | u | u | u | u | u | u | 
POP[A[D]] | u | u | u | u | u | u | 
CBW/CWDE/CWQE | u | u | u | u | u | u | 
MOVSX[D]/MOVZX[D] | u | u | u | u | u | u | 
ADD | 1 if bit carried out of operand, else 0 | 1 if operands had same sign and result has different sign, else 0 | same as sign bit of result | 1 if result is 0, else 0 | 1 if even number of 1's in lowest byte of result | 1 if carry out of bit 3 of result, else 0? | 
ADC | like ADD | like ADD | like ADD | like ADD | like ADD | like ADD | 
ADCX | like ADD | u | u | u | u | u | 
SUB | 1 if bit borrowed out of operand, else 0 | like ADD | like ADD | like ADD | like ADD | like ADD | 
SBB | like SUB | like SUB | like SUB | like SUB | like SUB | like SUB | 
INC/DEC | u | like ADD | like ADD | like ADD | like ADD | like ADD | 
CMP | like SUB | like SUB | like SUB | like SUB | like SUB | like SUB | 
NEG | 0 if operand is 0, else 1 | like SUB | like SUB | like SUB | like SUB | like SUB | 
MUL/IMUL | 0 if upper half of result is 0, else 1 | 0 if upper half of result is 0, else 1 | undef | undef | undef | undef | 
DIV/IDIV | undef | undef | undef | undef | undef | undef | 
DAA/DAS | 1 if decimal carry/borrow occurred, else 0 | undef | like ADD | like ADD | like ADD | 1 if decimal carry/borrow occurred, else 0 | 
AAA/AAS | like DAA/DAS | undef | undef | undef | undef | like DAA/DAS | 
AAM/AAD | undef | undef | like ADD for value in AL | like ADD for value in AL | like ADD for value in AL | undef | 
AND/OR/XOR | 0 | 0 | like ADD | like ADD | like ADD | undef | 
NOT | u | u | u | u | u | u | 
SAL/SHL | last bit shifted out of dest, if shift > 0, else u | only defined for 1-bit shifts: 0 if most sig bit of result = CF, else 0 | like ADD if shift > 0, else u | like ADD if shift > 0, else u | like ADD if shift > 0, else u | if shift > 0, undef, else u | 
SHR | like SAL | only defined for 1-bit shifts: most sig bit of original operand | like SAL | like SAL | like SAL | like SAL | 
SAR | like SAL | only defined for 1-bit shifts: 0 | like SAL | like SAL | like SAL | like SAL | 
SHLD/SHRD | like SAL | only defined for 1-bit shifts: 1 if sign change, else 0 | like SAL | like SAL | like SAL | like SAL | for shifts, all flags undefined if shift count > operand size
RCL/ROL | like SAL | only defined for 1-bit rots: most sig bit of result XOR CF after rot | u | u | u | u | 
RCR/ROR | like SAL | only defined for 1-bit rots: XOR of two most sig bits of result | u | u | u | u | 
BT/BTS/BTR/BTC | selected bit | undef | undef | u | undef | undef | 
BSF/BSR | undef | undef | undef | 1 if src = 0, else 0 | undef | undef | 
SETcc | u | u | u | u | u | u | 
TEST | 0 | 0 | like ADD | like ADD | like ADD | undef | 
JMP | only affected if task switch | only affected if task switch | only affected if task switch | only affected if task switch | only affected if task switch | only affected if task switch | non-status flags affected too if task switch
CALL | like JMP | like JMP | like JMP | like JMP | like JMP | like JMP | 
RET | u | u | u | u | u | u | 
IRET | restored to pre-interrupt value | restored to pre-interrupt value | restored to pre-interrupt value | restored to pre-interrupt value | restored to pre-interrupt value | restored to pre-interrupt value | 
Jcc | u | u | u | u | u | u | 
LOOP/LOOPcc | u | u | u | u | u | u | 
INT/INTO | may be replaced by task switch | may be replaced by task switch | may be replaced by task switch | may be replaced by task switch | may be replaced by task switch | may be replaced by task switch | 
BOUND | u | u | u | u | u | u | 
MOVS[B/W/D/Q] | u | u | u | u | u | u | 
CMP[B/W/D/Q] | like CMP | like CMP | like CMP | like CMP | like CMP | like CMP | 
SCAS[B/W/D/Q] | like CMP | like CMP | like CMP | like CMP | like CMP | like CMP | 
LODS[B/W/D/Q] | u | u | u | u | u | u | 
STOS[B/W/D/Q] | u | u | u | u | u | u | 
IN/INS/OUT/OUTS | u | u | u | u | u | u | 
ENTER/LEAVE | u | u | u | u | u | u | 
STC | 1 | u | u | u | u | u | 
CLC | 0 | u | u | u | u | u | 
CMC | not CF | u | u | u | u | u | 
STD | u | u | u | u | u | u | DF = 1
CLD | u | u | u | u | u | u | DF = 0
LAHF | u | u | u | u | u | u | 
SAHF | value from AH | u | value from AH | value from AH | value from AH | value from AH | 
PUSHF/PUSHFD | u | u | u | u | u | u | 
POPF/POPFD | value from stack | value from stack | value from stack | value from stack | value from stack | value from stack | DF and some system flags also take value from stack
STI | u | u | u | u | u | u | IF = 1
CLI | u | u | u | u | u | u | IF = 0
LEA | u | u | u | u | u | u | 
XLAT/XLATB | u | u | u | u | u | u | 
CPUID | u | u | u | u | u | u | 
NOP | u | u | u | u | u | u | 
RDRAND/RDSEED | 1 if random value available, else 0 | u | u | u | u | u | 
