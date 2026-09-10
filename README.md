# NPU Accelerator — RTL Design & Hardware Architecture

A custom **Neural Processing Unit (NPU) accelerator** designed from the ground up using **digital hardware design and RTL methodologies**.

This project explores how neural-network computations can be mapped onto dedicated hardware to achieve higher computational efficiency than a general-purpose processor. The design is developed incrementally, beginning with fundamental RTL building blocks and progressing toward a configurable matrix/compute accelerator suitable for neural-network workloads.

The primary focus of this project is not simply to create a working piece of RTL, but to understand the complete hardware-design flow:

**Architecture → RTL → Simulation → Verification → Synthesis → Physical Design Exploration**

---

## Overview

Modern AI workloads rely heavily on large numbers of repetitive mathematical operations, particularly **multiply-accumulate (MAC)** operations used in matrix multiplication and convolution.

General-purpose CPUs can execute these operations, but dedicated hardware accelerators can exploit the predictable structure of neural-network workloads by providing:

* Parallel computation
* High data reuse
* Reduced instruction overhead
* Specialized datapaths
* Efficient memory movement
* Lower energy per operation
* Higher throughput for AI workloads

This project aims to implement a simplified NPU architecture that demonstrates these principles at the RTL level.

The accelerator is designed around a configurable compute datapath capable of performing operations that form the basis of neural-network inference.

---

# Project Goals

The main goals of this project are:

1. Understand the architecture of dedicated AI accelerators.
2. Design the accelerator using synthesizable Verilog/SystemVerilog RTL.
3. Build the hardware incrementally from fundamental digital components.
4. Implement and verify arithmetic compute units.
5. Develop a parallel compute architecture.
6. Explore data movement and operand reuse.
7. Create comprehensive RTL testbenches.
8. Verify functionality through simulation.
9. Synthesize the design and analyze hardware characteristics.
10. Explore the transition from RTL to physical implementation.
11. Maintain a professional hardware-project structure suitable for a VLSI portfolio.

---

# Architecture

The NPU is designed as a collection of hardware modules rather than as one monolithic block.

At a high level, the architecture consists of:

```text
                    ┌───────────────────────────┐
                    │        Control Unit       │
                    │                           │
                    │  Configuration / Control  │
                    └─────────────┬─────────────┘
                                  │
                                  ▼
┌──────────────┐       ┌───────────────────────────┐
│ Input Buffer │──────▶│                           │
└──────────────┘       │       Compute Engine      │
                       │                           │
┌──────────────┐       │  ┌─────┐ ┌─────┐ ┌─────┐ │
│ Weight Buffer│──────▶│  │ MAC │ │ MAC │ │ MAC │ │
└──────────────┘       │  └─────┘ └─────┘ └─────┘ │
                       │                           │
                       │  ┌─────┐ ┌─────┐ ┌─────┐ │
                       │  │ MAC │ │ MAC │ │ MAC │ │
                       │  └─────┘ └─────┘ └─────┘ │
                       │                           │
                       └─────────────┬─────────────┘
                                     │
                                     ▼
                            ┌────────────────┐
                            │ Output Buffer  │
                            └────────────────┘
```

The exact architecture will evolve as the implementation progresses.

---

# Compute Engine

The central component of the NPU is the **compute engine**.

Neural-network layers can frequently be expressed using matrix operations such as:

$$
C = A \times B
$$

For individual output elements:

$$
C_{ij} = \sum_k A_{ik}B_{kj}
$$

This operation requires repeated multiplication followed by addition.

The fundamental hardware operation is therefore:

$$
ACC_{new} = ACC_{old} + A \times B
$$

This is known as a **Multiply-Accumulate (MAC)** operation.

A MAC unit forms the basic computational element of the accelerator.

---

# MAC Unit

The MAC unit is responsible for performing:

```text
          ┌─────────┐
Input A ─▶│         │
          │Multiplier├──────┐
Input B ─▶│         │      │
          └─────────┘      ▼
                         ┌─────┐
Accumulator ────────────▶│ ADD │
                         └──┬──┘
                            │
                            ▼
                       Accumulator
```

Instead of calculating each multiplication sequentially using software instructions, multiple MAC units can operate concurrently.

For example, a compute engine containing multiple MAC units can perform:

```text
MAC 0 ──▶ A0 × B0
MAC 1 ──▶ A1 × B1
MAC 2 ──▶ A2 × B2
MAC 3 ──▶ A3 × B3
       ...
```

This parallelism is one of the fundamental advantages of an accelerator architecture.

---

# Parallelism

A major objective of the NPU is to exploit **spatial parallelism**.

Rather than relying on a single arithmetic unit:

```text
A ──▶ MAC ──▶ Result
```

the accelerator can contain an array of processing elements:

```text
             ┌─────┐
        ┌───▶│ MAC │───┐
        │    └─────┘   │
        │              │
Input ──┼───▶┌─────┐   ├──▶ Output
        │    │ MAC │───┤
        │    └─────┘   │
        │              │
        └───▶┌─────┐   │
             │ MAC │───┘
             └─────┘
```

As the architecture develops, this can evolve toward a structured compute array.

The objective is to increase the number of useful operations performed per clock cycle while keeping the hardware manageable and synthesizable.

---

# Dataflow

Hardware acceleration is not only about adding more arithmetic units.

A significant portion of accelerator performance depends on **how data moves through the hardware**.

Moving data between memory and compute units can consume significant time and energy.

Therefore, the architecture explores concepts such as:

* Input reuse
* Weight reuse
* Partial-sum reuse
* Local buffering
* Parallel data movement
* Structured processing-element communication

The design can eventually be extended toward dataflow architectures such as:

* Weight-stationary
* Output-stationary
* Input-stationary
* Systolic-style data movement

The initial implementation prioritizes understanding the fundamental principles before introducing additional architectural complexity.

---

# Numeric Representation

Neural-network accelerators frequently use reduced-precision arithmetic instead of conventional high-precision floating-point computation.

Possible representations include:

* INT8
* INT16
* INT32
* Fixed-point formats

The initial implementation uses integer arithmetic to keep the datapath understandable and synthesizable.

The project may later investigate reduced-precision computation and the corresponding hardware trade-offs.

Important considerations include:

* Accumulator width
* Overflow
* Signed arithmetic
* Quantization
* Precision
* Hardware area
* Timing
* Power

---

# RTL Design

The accelerator is implemented using synthesizable RTL.

The RTL describes the hardware structure and behavior at the register-transfer level.

The design is organized into modular components so that individual blocks can be developed and verified independently.

Example hierarchy:

```text
NPU
│
├── Control Unit
│
├── Input Buffer
│
├── Weight Buffer
│
├── Compute Engine
│   │
│   ├── Processing Element
│   │
│   ├── Multiplier
│   │
│   ├── Accumulator
│   │
│   └── MAC Array
│
├── Output Buffer
│
└── Interface Logic
```

The exact hierarchy will evolve as the design becomes more sophisticated.

---

# Development Philosophy

The project follows a **bottom-up hardware-design methodology**.

Instead of attempting to write the complete NPU in one step, the design is developed through progressively more complex blocks.

### Stage 1 — Digital Fundamentals

Build and verify basic RTL components.

Examples:

* Logic gates
* Combinational logic
* Sequential logic
* Registers
* Counters
* Multiplexers
* Basic control logic

### Stage 2 — Arithmetic Hardware

Develop arithmetic datapaths.

Examples:

* Adders
* Multipliers
* Accumulators
* Signed arithmetic
* MAC units

### Stage 3 — Processing Element

Combine arithmetic components into a reusable processing element.

### Stage 4 — Parallel Compute

Instantiate multiple processing elements to create a compute array.

### Stage 5 — Data Movement

Introduce buffering and structured movement of operands and partial sums.

### Stage 6 — Control

Implement the control logic required to coordinate computation.

### Stage 7 — NPU Integration

Integrate the individual modules into the complete accelerator.

### Stage 8 — Verification

Develop increasingly comprehensive testbenches and functional checks.

### Stage 9 — Synthesis

Synthesize the RTL and analyze:

* Area
* Timing
* Cell utilization
* Critical paths
* Logic depth

### Stage 10 — Physical Design Exploration

Where supported by the available toolchain, investigate:

**RTL → Synthesis → Floorplanning → Placement → Routing**

This provides exposure to the broader ASIC design flow.

---

# Verification

Verification is treated as an essential part of the project rather than an afterthought.

Each module is tested independently before integration.

The verification process includes:

* Directed test cases
* Boundary conditions
* Reset behavior
* Arithmetic correctness
* Signed/unsigned behavior
* Overflow conditions
* Multiple input combinations
* Sequential behavior
* Integration testing

Example:

```text
Test Input
     │
     ▼
┌──────────────┐
│ RTL Design   │
└──────┬───────┘
       │
       ▼
Simulation
       │
       ▼
Expected Result
       │
       ▼
   PASS / FAIL
```

Waveforms are used to inspect internal signals and verify that the hardware behaves as intended over time.

---

# Simulation

RTL simulation is performed before synthesis.

Simulation allows the design to be tested without physically fabricating or implementing the circuit.

The project uses a simulation workflow based around open-source RTL tools.

Typical workflow:

```text
Verilog/SystemVerilog
        │
        ▼
     Compile
        │
        ▼
   Testbench
        │
        ▼
    Simulation
        │
        ▼
     Waveform
        │
        ▼
    Verification
```

Simulation output and waveform screenshots are maintained as evidence of functional verification.

---

# Synthesis

After functional verification, the RTL can be synthesized into a gate-level representation.

The synthesis process transforms behavioral RTL into hardware structures based on a target technology library.

Conceptually:

```text
RTL
 │
 ▼
Elaboration
 │
 ▼
Logic Synthesis
 │
 ▼
Technology Mapping
 │
 ▼
Gate-Level Netlist
```

The synthesized design can then be analyzed for:

* Area
* Timing
* Logic utilization
* Critical paths
* Number of sequential elements
* Number of combinational cells

These results help evaluate whether architectural decisions are producing efficient hardware.

---

# Physical Design Exploration

A longer-term objective of the project is to explore the path from synthesized RTL toward physical implementation.

The general ASIC flow is:

```text
RTL
 │
 ▼
Synthesis
 │
 ▼
Netlist
 │
 ▼
Floorplan
 │
 ▼
Placement
 │
 ▼
Clock Tree Synthesis
 │
 ▼
Routing
 │
 ▼
Physical Verification
 │
 ▼
GDSII
```

This stage provides practical exposure to concepts such as:

* Standard cells
* Floorplanning
* Placement density
* Routing congestion
* Clock distribution
* Timing closure
* Physical area
* Design-rule constraints

The physical implementation stage depends on the maturity of the RTL and the available open-source technology/toolchain.

---

# Project Structure

The repository is organized to separate RTL, software, verification, synthesis, and documentation.

```text
Vikrant-A1/
│
├── docs/
│   ├── architecture/
│   ├── diagrams/
│   └── screenshots/
│
├── rtl/
│   ├── common/
│   ├── arithmetic/
│   ├── processing_element/
│   ├── compute_engine/
│   ├── buffers/
│   ├── control/
│   └── npu_top/
│
├── verification/
│   ├── testbenches/
│   ├── test_vectors/
│   └── waveforms/
│
├── synthesis/
│   ├── scripts/
│   ├── reports/
│   └── netlists/
│
├── software/
│   ├── reference_model/
│   └── test_data/
│
└── README.md
```

The repository structure may change as additional functionality is implemented.

---

# Reference Model

A software reference model can be used to establish the mathematically correct result before comparing it with the RTL implementation.

For example:

```text
Software Model
      │
      │ Expected Result
      ▼
   ┌───────┐
   │       │
   │Compare│
   │       │
   └───┬───┘
       ▲
       │ Actual Result
       │
      RTL
```

This allows the hardware implementation to be checked against an independent reference.

For matrix operations, the reference model can calculate the expected output using conventional software arithmetic.

---

# Performance Metrics

As the project develops, several metrics will be tracked.

## Functional Metrics

* Correctness
* Test coverage
* Number of verified modules
* Number of passing test cases

## Hardware Metrics

* Area
* Gate count
* Number of registers
* Number of arithmetic units
* Critical-path delay
* Maximum estimated frequency

## Accelerator Metrics

* Operations per cycle
* Throughput
* MAC utilization
* Memory accesses
* Data reuse
* Compute efficiency

One important objective is to understand the relationship between architecture and these hardware metrics rather than optimizing only for raw computational throughput.

---

# Design Trade-offs

A hardware accelerator involves multiple competing constraints.

Increasing parallelism can improve throughput, but it also increases:

* Hardware area
* Data movement
* Routing complexity
* Power consumption

Similarly, increasing precision improves numerical accuracy but generally requires wider datapaths and larger hardware.

The project therefore examines trade-offs such as:

```text
Performance
     ▲
     │
     │       ●
     │    ●
     │ ●
     └──────────────────▶
       Area / Complexity
```

The goal is not simply to maximize one metric, but to understand how architectural decisions affect the complete system.

---

# Current Status

The project is currently under active development.

### Completed

* Initial repository structure
* Basic RTL development environment
* Fundamental RTL experimentation
* Basic simulation workflow
* Initial logic-gate verification
* Simulation output generation

### In Progress

* Arithmetic datapath development
* MAC architecture
* Processing-element design
* Compute-engine architecture
* Verification infrastructure

### Planned

* Parallel MAC array
* Input/weight buffering
* Control architecture
* Matrix computation
* More extensive verification
* Synthesis
* Timing and area analysis
* Physical-design exploration

The project is intentionally developed incrementally so that each architectural stage can be understood and verified before moving to the next.

---

# Example Development Flow

A typical development cycle is:

```text
        ┌───────────────┐
        │ Define Block  │
        └───────┬───────┘
                ▼
        ┌───────────────┐
        │ Design RTL    │
        └───────┬───────┘
                ▼
        ┌───────────────┐
        │ Write Testbench│
        └───────┬───────┘
                ▼
        ┌───────────────┐
        │ Run Simulation │
        └───────┬───────┘
                ▼
          ┌───────────┐
          │   PASS?   │
          └─────┬─────┘
             NO │ YES
                │
        ┌───────▼───────┐
        │ Debug / Fix   │
        └───────┬───────┘
                │
                └───────────────┐
                                ▼
                       ┌────────────────┐
                       │ Integrate Block│
                       └───────┬────────┘
                               ▼
                           Synthesis
```

This workflow is repeated for each major component.

---

# Why Build an NPU?

Neural-network accelerators provide an excellent intersection between:

* Digital logic
* Computer architecture
* Semiconductor design
* Parallel computing
* Embedded systems
* Artificial intelligence

Building an NPU therefore provides an opportunity to study both **how AI algorithms work** and **how specialized silicon executes those algorithms efficiently**.

The project is intended to bridge the gap between software-level AI concepts and physical hardware implementation.

---

# Learning Objectives

Through this project, the following concepts are being explored:

### Digital Design

* Combinational logic
* Sequential logic
* Registers
* FSMs
* Counters
* Datapaths

### Computer Architecture

* Parallelism
* Pipelining
* Buffers
* Dataflow
* Memory hierarchy
* Compute architectures

### AI Hardware

* Matrix multiplication
* MAC operations
* Processing elements
* Neural-network inference
* Quantization
* Data reuse

### VLSI

* RTL design
* Simulation
* Synthesis
* Standard cells
* Timing analysis
* Area analysis
* Physical design

---

# Future Improvements

Potential future extensions include:

* Larger MAC arrays
* Pipelined datapaths
* Configurable matrix dimensions
* Multiple precision modes
* INT8 inference
* Activation functions
* ReLU hardware
* Bias addition
* On-chip buffering
* DMA-style data movement
* Improved control logic
* Systolic-array architecture
* Performance counters
* Hardware/software co-simulation
* FPGA implementation
* ASIC physical implementation
* Power estimation

The architecture will be extended only after the underlying components are properly verified.

---

# Engineering Principles

This project follows several principles throughout development:

### 1. Verify Before Integrating

A block should be tested independently before becoming part of a larger subsystem.

### 2. Keep Modules Reusable

Hardware components should have clearly defined interfaces and limited responsibilities.

### 3. Prefer Measurable Results

Claims about performance or efficiency should be supported by simulation, synthesis, or measured results.

### 4. Document Architectural Decisions

Important design decisions should be recorded along with their reasoning and trade-offs.

### 5. Build Incrementally

Complex hardware should be constructed from smaller, understandable components.

---

# Evidence & Documentation

The repository maintains development evidence including:

* RTL source
* Testbenches
* Simulation output
* Waveforms
* Architecture diagrams
* Synthesis reports
* Timing reports
* Area reports
* Implementation screenshots

This documentation is intended to make the development process reproducible and auditable.

---

# Tools & Technologies

The project uses an open-source-oriented hardware development workflow.

Current and planned tools include:

* **Verilog/SystemVerilog** — RTL design
* **Icarus Verilog** — RTL simulation
* **GTKWave** — waveform analysis
* **Synthesis tools** — RTL-to-gate conversion
* **Open-source ASIC tools** — physical-design exploration
* **SkyWater SKY130** — potential open-source technology target

The exact toolchain may evolve as the project progresses.

---

# Repository Philosophy

This repository is not intended to represent a finished commercial NPU.

Instead, it documents the process of designing one from fundamental digital hardware concepts toward a complete accelerator architecture.

The emphasis is on:

**Understanding → Designing → Simulating → Verifying → Synthesizing → Measuring**

rather than simply producing a large amount of RTL.

---

# Author

**Vikrant**

Independent hardware-design project focused on exploring:

**VLSI • Computer Architecture • AI Accelerators • Semiconductor Engineering**

---

# Project Status

🚧 **Active Development**

The architecture, RTL, verification environment, and implementation flow are continuously evolving.

Future commits will progressively move the project from fundamental RTL components toward a complete neural-network accelerator.

---

## Final Objective

The long-term objective is to produce a complete, documented hardware accelerator capable of executing meaningful neural-network computations while demonstrating the complete digital-design workflow:

```text
        Neural Network
              │
              ▼
       Mathematical Model
              │
              ▼
        NPU Architecture
              │
              ▼
            RTL
              │
              ▼
          Simulation
              │
              ▼
         Verification
              │
              ▼
          Synthesis
              │
              ▼
      Gate-Level Netlist
              │
              ▼
        Physical Design
              │
              ▼
       Hardware Layout
```

This project serves as a practical exploration of how an idea at the algorithmic level can eventually become a piece of digital silicon.
