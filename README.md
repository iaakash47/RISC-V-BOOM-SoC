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
* Fetch - Instructions are fetched from instruction memory and pushed into a FIFO queue, known as the Fetch Buffer.
* Decode - Decode pulls instructions out of the Fetch Buffer and generates the appropriate Micro-Ops to place into the pipeline. Here uOPs is called as Micro Operations
* Rename - The ISA, or logical, register specifiers are then renamed into physical register specifiers.
* Issue - UOPs sitting in a Issue Queue wait until all of their operands are ready and are then issued.
* Register Read - Issued UOPs first read their register operands from the unified Physical Register File 
* Execute - Execute stage where the functional units reside. Issued memory operations perform their address calculations in the Execute stage, and then     store the calculated addresses in the Load/Store Unit which resides in the Memory stage.

* Memory - The Load/Store Unit consists of three queues: a Load Address Queue (LAQ), a Store Address Queue (SAQ), and a Store Data Queue (SDQ). Loads are   fired to memory when their address is present in the LAQ. Stores are fired to memory at Commit time

* Writeback - ALU operations and load operations are written back to the Physical Register File.

# How Does RISC-V BOOM CPU Cache Work and What Are L1, L2 Cache?
* Cache is just a really fast type of memory and the CPU cache stands at the top of this hierarchy, being the fastest,cache memory operates between 10 to   100 times faster than RAM
* CPU Cache memory is divided into three levels: L1, L2, and L3
  L1 (Level 1) cache is the fastest memory that is present in a computer system/BOOM system. In terms of priority of access, the L1 cache has the data     the CPU is most likely to need while completing a certain task.
  The L1 cache is usually split into two sections: the instruction cache and the data cache. The instruction cache deals with the information about the     operation that the CPU must perform, while the data cache holds the data on which the operation is to be performed.
  
  L2 (Level 2) cache is slower than the L1 cache but bigger in size. Where an L1 cache may measure in kilobytes, modern L2 memory caches measure in 512kb   in Boom SoC (kilobytes to megabytes) The L2 cache size varies depending on the CPU, but its size is typically between 256KB to 32MB and When it comes     to speed, the L2 cache lags behind the L1 cache but is still much faster than your system RAM
