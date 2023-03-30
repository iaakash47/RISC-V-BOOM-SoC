# RISC-V-BOOM-SoC

SystemVerilog DPI (Direct Programming Interface) is an interface which can be used to interface SystemVerilog with foreign languages. These Foreign languages can be C, C++, SystemC as well as others.
DPI allows the user to easily call functions of other language from SystemVerilog and to export SystemVerilog functions, so that they can be called in other languages.
Advantage of DPI is,  it allows to make use of the code exists in other languages.

Import Method
The methods (functions/tasks) implemented in Foreign language can be called from SystemVerilog and such methods are called Import methods.

Export methods
The methods implemented in SystemVerilog can be called from Foreign language such methods are called Export methods.


#BOOM System on Chip

The Berkeley Out-of-Order Machine (BOOM) is a synthesizable and parameterizable open-source RISC-V out-of-order core written in the Chisel hardware construction language. Because Chisel does not support blocking assignments, it is impossible to create a deterministic divided clock.

# The BOOM Pipeline
BOOM is broken up into 10 stages: Fetch, Decode, Register Rename, Dispatch, Issue, Register Read, Execute, Memory, Writeback and Commit. 
Many of those stages are combined in the current implementation, yielding seven stages: Fetch, Decode/Rename, Rename/Dispatch, Issue/RegisterRead, Execute, Memory and Writeback stages of pipleine.


## Stages
# Fetch - Instructions are fetched from instruction memory and pushed into a FIFO queue, known as the Fetch Buffer.
# Decode - Decode pulls instructions out of the Fetch Buffer and generates the appropriate Micro-Ops to place into the pipeline. Here uOPs is called as Micro Operations
# Rename - The ISA, or logical, register specifiers are then renamed into physical register specifiers.
# Issue - UOPs sitting in a Issue Queue wait until all of their operands are ready and are then issued.
# Register Read - Issued UOPs first read their register operands from the unified Physical Register File 
Execute - Execute stage where the functional units reside. Issued memory operations perform their address calculations in the Execute stage, and then store the calculated addresses in the Load/Store Unit which resides in the Memory stage.

Memory - The Load/Store Unit consists of three queues: a Load Address Queue (LAQ), a Store Address Queue (SAQ), and a Store Data Queue (SDQ). Loads are fired to memory when their address is present in the LAQ. Stores are fired to memory at Commit time

Writeback - ALU operations and load operations are written back to the Physical Register File.
