# GQH Hardware Track — Code Submission & Participant Resources

This repository contains the **official hardware-track code-submission instructions, judging/testing references, recommended repository structure, and README template** for teams participating in the GQH FPGA Trade Signal Hackathon.

> **Important:** Each team keeps its **own GitHub repository** for its project. This GQH repository contains shared instructions and organizer-provided resources; teams do not upload all project code here.

## Official Submission Channel

**All final hackathon submissions will be made through Devpost.**

Teams should place their project code in their own GitHub repository and include the repository link in the official Devpost submission.

Before the deadline, each team should:

1. Build and test the final FPGA design.
2. Push the final source/project files to the team's own GitHub repository.
3. Make sure the repository is accessible to judges.
4. Create a final commit and record its **full Git commit SHA**.
5. Complete the official Devpost submission with the required project information, GitHub repository, video/materials, and final commit SHA if a dedicated field is provided.
6. Review [SUBMISSION_CHECKLIST.md](SUBMISSION_CHECKLIST.md).

## Start Here

- [Judging & Testing Rules](JUDGING_AND_TESTING.md)
- [Team README Template](TEAM_README_TEMPLATE.md)
- [Recommended Repository Structure](REPOSITORY_STRUCTURE.md)
- [Final Submission Checklist](SUBMISSION_CHECKLIST.md)
- [Participant Testing Resources](participant-resources/README.md)

## Organizer-Provided Testing References

The following reference guides are available in this repository:

- [Quick UART Test Reference](participant-resources/testing/21_quick_uart_test_REFERENCE.md)
- [Robust UART / Scoring Test Reference](participant-resources/testing/22_robust_uart_test_REFERENCE.md)

The corresponding Python testing scripts and organizer-supplied Tang Nano 20K `.cst` constraint file will be placed in the participant-resources folder when they are distributed by the organizers.

## Judging Criteria

Hardware-track judging uses three technical criteria:

1. **Correctness of outputs** during the official **1000-packet** judging run.
2. **Input-to-output response delay / latency** measured by the official UART test.
3. **LUT usage** reported by Gowin after synthesis.

See [JUDGING_AND_TESTING.md](JUDGING_AND_TESTING.md) for the exact packet interface, latency definition, LUT-report instructions, and testing workflow.

## Fixed Packet Protocol

The external interface is fixed and must not be changed.

### PC → FPGA

```text
[index16][item1_8][price1_16][item2_8][price2_16]
```

### FPGA → PC

```text
[index16][item1_8][action1_8][item2_8][action2_8][reserved16]
```

### Fixed IDs

```text
ITEM_A = 0x11 = 00010001
ITEM_B = 0x22 = 00100010

NONE = 0x00 = 00000000
SELL = 0x01 = 00000001
BUY  = 0x02 = 00000010
```

Both packets are **64 bits / 8 bytes**. Multi-byte fields are transmitted **big-endian**, and UART operates at **115200 baud**.

Do not modify the packet layout, field widths, byte order, item IDs, action IDs, or packet length.

## Organizer-Supplied Constraint File

Participants **do not need to create a new pin-constraint file in FloorPlanner**.

The organizers will provide the Tang Nano 20K `.cst` constraint file. Add the supplied file to the Gowin project as the physical constraint file and make sure the top-level port names match the supplied constraints.

## What the Team Repository Should Contain

At minimum:

- Final HDL/VHDL source code
- The organizer-supplied Tang Nano 20K `.cst` file
- Gowin project/build files needed to reproduce the design
- A complete `README.md`
- The top-level entity/module name
- Toolchain/version information
- Any host-side code required by the team project
- A disclosure of external libraries, IP cores, starter code, or other pre-existing resources used

Include testbenches, generated bitstreams, benchmark files, or other artifacts when applicable or required by organizers.

## Final Commit SHA

Once the final project is ready:

```bash
git status
git add .
git commit -m "Final hackathon submission"
git push
git rev-parse HEAD
```

Save the entire SHA printed by the final command. This identifies the exact version intended for judging.

## Repository Access

Teams are responsible for ensuring judges can access the submitted repository.

- **Public repository:** verify that the URL opens normally.
- **Private repository:** follow the organizer-provided judging-access instructions before the deadline.

Never commit passwords, API keys, access tokens, private keys, or other secrets.

## Devpost

The official Devpost URL and final deadline will be added once confirmed by the organizers.

**Devpost:** _To be added_  
**Deadline:** _To be added_

---

For event-specific questions, use the official GQH Hardware Track communication channel.
