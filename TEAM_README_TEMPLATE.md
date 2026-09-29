# Project Name

## Team Members

- Name — Email
- Name — Email
- Name — Email
- Name — Email

## Project Overview

Briefly explain what your project does, the problem it addresses, and the main idea behind your solution.

## FPGA Implementation

Describe **what runs directly on the FPGA**.

Be specific about the computations, algorithms, signal processing, market-data processing, control logic, or other functionality implemented in hardware.

Also describe any functionality that runs on a host computer rather than on the FPGA.

## Hardware

- FPGA board: Tang Nano 20K
- FPGA number assigned to team: `#___`
- Additional hardware/peripherals used:

## HDL / Languages

- HDL used:
- Host-side language(s), if applicable:

## Toolchain

- Gowin EDA version:
- Other required software/tools:
- Operating system used for development, if relevant:

## Top-Level Entity / Module

```text
top_level_name_here
```

## Repository Structure

Briefly describe the important folders/files in this repository.

Example:

```text
src/          HDL source files
constraints/  Tang Nano 20K .cst constraints
testbench/    simulation/testbench files
gowin/        Gowin project/build files
host/         optional host-side software
bitstream/    generated bitstream, if included
results/      optional benchmark/output files
```

## Build Instructions

Provide enough detail for an organizer or judge to reproduce the FPGA build.

1. Open/install the required toolchain.
2. Open/import the project.
3. Set the top-level entity/module.
4. Add the required source and constraint files.
5. Run synthesis.
6. Run place & route.
7. Generate the bitstream/programming file.
8. Note any additional required steps.

Include any project-specific settings that are necessary.

## Programming the Tang Nano 20K

Explain how to load the final design onto the board.

1. Connect the Tang Nano 20K.
2. Open the required programming tool.
3. Select the generated programming file.
4. Program the board.
5. Verify expected behavior.

Add any project-specific steps.

## Inputs

Describe the inputs to the FPGA/project.

Examples:

- Buttons or switches
- UART/serial data
- Market data
- Host-computer input
- Test vectors
- Clock/reset behavior

## Outputs

Describe the expected outputs.

Examples:

- LEDs
- UART/serial output
- Display output
- Signals returned to a host computer
- Benchmark/result files

## How to Reproduce the Demo

Give a short step-by-step procedure that reproduces what your team demonstrated during the hackathon.

1.
2.
3.

Expected result:

```text
Describe what the judge should observe.
```

## Verification / Testing

Describe how your team tested the final design.

If applicable, include:

- Testbench instructions
- Simulation results
- Hardware verification procedure
- Known edge cases
- Benchmark methodology

## Performance / Results

If applicable, report relevant results such as:

- Latency
- Throughput
- Clock frequency
- FPGA resource utilization
- Quant/strategy performance metrics
- Other measurements used in your project

Explain how each result was measured.

## External Libraries / IP / Starter Code

List any external libraries, IP cores, starter code, open-source projects, datasets, or other pre-existing resources used.

For each item, include:

- Name
- Source/link
- Purpose in the project

If none were used, write:

```text
None.
```

## Known Limitations

Document any limitations, incomplete features, assumptions, or known issues that judges should be aware of.

## Final Submission

- GitHub repository URL:
- Final commit SHA:
- Demo video URL, if applicable:
- Devpost URL, if applicable:
