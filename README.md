# RISC-V-BOOM-SoC

SystemVerilog DPI (Direct Programming Interface) is an interface which can be used to interface SystemVerilog with foreign languages. These Foreign languages can be C, C++, SystemC as well as others.
DPI allows the user to easily call functions of other language from SystemVerilog and to export SystemVerilog functions, so that they can be called in other languages.
Advantage of DPI is,  it allows to make use of the code exists in other languages.

Import Method
The methods (functions/tasks) implemented in Foreign language can be called from SystemVerilog and such methods are called Import methods.

Export methods
The methods implemented in SystemVerilog can be called from Foreign language such methods are called Export methods.


## BOOM System on Chip Architecture

The Berkeley Out-of-Order Machine (BOOM) is a synthesizable and parameterizable open-source RISC-V out-of-order core written in the Chisel hardware construction language. Because Chisel does not support blocking assignments, it is impossible to create a deterministic divided clock.

![boom-pipeline-detailed](https://user-images.githubusercontent.com/88897605/229101205-d68503f4-b6b9-4db0-904b-bc45e3bb90fd.jpg)


## The BOOM Pipeline
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
* Cache is just a really fast type of memory and the CPU cache stands at the top of this hierarchy, being the fastest,cache memory operates between 10 to 100 times faster than RAM
* CPU Cache memory is divided into three levels: L1, L2, and L3
* L1 (Level 1) cache is the fastest memory that is present in a computer system/BOOM system. In terms of priority of access, the L1 cache has the data the CPU is most likely to need while completing a certain task.
  The L1 cache is usually split into two sections: the instruction cache and the data cache. The instruction cache deals with the information about the operation that the CPU must perform, while the data cache holds the data on which the operation is to be performed.
  
* L2 (Level 2) cache is slower than the L1 cache but bigger in size. Where an L1 cache may measure in kilobytes, modern L2 memory caches measure in 512kb in Boom SoC (kilobytes to megabytes) The L2 cache size varies depending on the CPU, but its size is typically between 256KB to 32MB and When it comes to speed, the L2 cache lags behind the L1 cache but is still much faster than your system RAM


## Rocket Chip SoC Generator Architecture 
![chip](https://user-images.githubusercontent.com/88897605/229102201-44804a08-e1c1-4595-a336-73b50c44833b.png)
* As BOOM is just a core, an entire SoC infrastructure must be provided. BOOM was developed to use the open-source Rocket Chip SoC generator. The Rocket Chip generator can instantiate a wide range of SoC designs, including cache-coherent multi-tile designs, cores with and without accelerators, and chips with or without a last-level shared cache. It comes bundled with a 5-stage in-order core, called Rocket, by default. BOOM uses the Rocket Chip infrastructure to instantiate it’s core/tile complex (tile is a core, L1D/I$, and PTW) instead of a Rocket tile.
* Rocket that are also used by BOOM - the functional units, the caches, the translation look-aside buffers (TLBs), the page table walker (PTW), and more

## The Rocket Core - a Library of Processor Components

From BOOM’s point of view, the Rocket core can be thought of as a “Library of Processor Components.” There are a number of modules created for Rocket that are also used by BOOM - the functional units, the caches, the translation look-aside buffers (TLBs), the Page Table Walker (PTW), and more. Throughout this document you will find references to these Rocket components and descriptions on how they fit into BOOM.


## Core Overview

### Instruction Fetch
![image](https://user-images.githubusercontent.com/88897605/229439211-04bc40c4-8b94-4615-aef1-9fabb4379413.png)
BOOM instantiates its own Front-end , similar to how the Rocket core(s) instantiates its own Front-end . This Front-end fetches instructions and makes predictions throughout the Fetch stage to redirect the instruction stream in multiple fetch cycles (F0, F1…). If a misprediction is detected in BOOM’s Back-end (execution pipeline), or one of BOOM’s own predictors wants to redirect the pipeline in a different direction, a request is sent to the Front-end and it begins fetching along a new instruction path.

### The Rocket Core I-Cache

BOOM instantiates the i-cache taken from the Rocket processor source code. The i-cache is a virtually indexed, physically tagged set-associative cache.
To save power, the i-cache reads out a fixed number of bytes (aligned) and stores the instruction bits into a register. Further instruction fetches can be managed by this register. The i-cache is only fired up again once the fetch register has been exhausted (or a branch prediction directs the PC elsewhere). The i-cache does not (currently) support fetching across cache-lines, nor does it support fetching unaligned relative to the superscalar fetch address. The i-cache does not (currently) support hit-under-miss. If an i-cache miss occurs, the i-cache will not accept any further requests until the miss has been handled. This is less than ideal for scenarios in which the pipeline discovers a branch mispredict and would like to redirect the i-cache to start fetching along the correct path.

#### The Fetch Buffer
Fetch Packet s coming from the i-cache are placed into a Fetch Buffer . The Fetch Buffer helps to decouple the instruction fetch Front-end from the execution pipeline in the Back-end .The Fetch Buffer is parameterizable. The number of entries can be changed and whether the buffer is implemented as a “flow-through” queue or not can be toggled.

###### Instruction Cache - An instruction cache is a cache that is accessed only as a result of an instruction fetch. Therefore, an instruction cache is never written to by any load or store instruction executed by the processor

#### Branch Prediction
BOOM uses two levels of branch prediction - a fast Next-Line Predictor (NLP) and a slower but more complex Backing Predictor (BPD). In this case, the NLP is a Branch Target Buffer and the BPD is a more complicated structure like a GShare predictor.

#### The Decode Stage
The Decode stage takes instructions from the Fetch Buffer, decodes them, and allocates the necessary resources as required by each instruction. The Decode stage will stall as needed if not all resources are available.

#### The Rename Stage
The Rename stage maps the ISA (or logical) register specifiers of each instruction to physical register specifiers.

###### The Purpose of Renaming
Renaming is a technique to rename the ISA (or logical) register specifiers in an instruction by mapping them to a new space of physical registers. The goal to register renaming is to break the output-dependencies (WAW) and anti-dependences (WAR) between instructions, leaving only the true dependences (RAW)
