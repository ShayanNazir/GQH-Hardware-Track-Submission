# Project Name

## Team Members

- Name — Email
- Name — Email
- Name — Email
- Name — Email

## Project Overview

Briefly explain what your project does and the main idea behind the implementation.

## FPGA Implementation

Describe what runs directly on the FPGA and any host-side functionality.

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
- Operating system, if relevant:

## Top-Level Entity / Module

```text
top_level_name_here
```

## Organizer-Supplied Constraint File

State that the organizer-provided Tang Nano 20K `.cst` file is used and identify its location in this repository.

Do not recreate the board pin constraints in FloorPlanner unless explicitly instructed by organizers.

## Repository Structure

Briefly describe the important folders/files.

## Build Instructions

1. Open/import the Gowin project.
2. Add/verify required source files.
3. Add the organizer-supplied `.cst` as the physical constraint file.
4. Verify the top-level entity/module.
5. Run synthesis.
6. Run Place & Route.
7. Generate the programming file.
8. Note any project-specific steps.

## Programming the Tang Nano 20K

Explain how to load the final design onto the board.

## Fixed UART Interface

Confirm that the design follows the official interface:

```text
PC -> FPGA:
[index16][item1_8][price1_16][item2_8][price2_16]

FPGA -> PC:
[index16][item1_8][action1_8][item2_8][action2_8][reserved16]

ITEM_A = 0x11
ITEM_B = 0x22

NONE = 0x00
SELL = 0x01
BUY  = 0x02

UART = 115200 baud
Packet size = 8 bytes each direction
Multi-byte fields = big-endian
```

## How to Reproduce the Demo

1.
2.
3.

Expected result:

```text
Describe what the judge should observe.
```

## Verification / Testing

Describe how the design was tested, including use of the organizer-provided UART testing scripts when applicable.

## Judging Metrics / Results

### Correctness

- Local test used:
- Correctness result:

> Official judging uses a 1000-packet run.

### Latency

- Average measured input-to-output round-trip latency:
- Test/setup used:

### LUT Usage

After synthesis, open:

**Synthesis Report → Resource → Resource Usage Summary**

Record:

- Total LUT:
- LUT2:
- LUT3:
- LUT4:
- Other relevant resource usage:

## External Libraries / IP / Starter Code

List any external resources used and their purpose. If none were used, write `None.`

## Known Limitations

Document any limitations, assumptions, or known issues.

## Final Submission

- GitHub repository URL:
- Final commit SHA:
- Demo video URL, if applicable:
- Devpost URL:
