# GQH Hardware Track — Code Submission & Participant Resources

This repository contains the **official hardware-track code-submission instructions, judging/testing references, recommended repository structure, and README template** for teams participating in the GQH FPGA Trade Signal Hackathon.

> **Source of truth:** the [GQH Hardware Track Participant Guide (PDF)](participant-resources/GQH_Hardware_Track_Participant_Guide.pdf). If anything in this repository disagrees with the guide, the guide wins.

> **Important:** Each team keeps its **own GitHub repository** for its project. This GQH repository contains shared instructions and organizer-provided resources; do not upload team project code here.

## Official Links & Dates

| | |
|---|---|
| **Devpost** | <https://gqhacks.devpost.com> |
| **Submission deadline** | **Sunday, October 4, 2026, 11:00 am EDT** |
| **Questions / support** | GQH Hardware Track Discord: [discord.gg/9qPMtN4UB](https://discord.gg/9qPMtN4UB) |

## Board Pickup & Drop-off

| | Pickup | Drop-off |
|---|---|---|
| **When** | Friday, October 2, 7:15 pm | By Sunday, October 4, 11:00 am |
| **Where** | Reitz Room 2345 | Reitz Room 2345 |
| **Requirement** | Team registered and FPGA Loan Agreement signed | Final code submitted to GitHub and Devpost; board and all accessories handed in |

Submit first, then hand in the board and accessories. Judges program each board in **SRAM mode** with the `.fs` file from your submitted commit and test it on one judging PC. Changes pushed after drop-off are not judged. See Part 1 of the [participant guide](participant-resources/GQH_Hardware_Track_Participant_Guide.pdf) for what to bring, check-out/check-in steps, and late, lost, or damaged boards.

## Minimum Viable Submission

Receive 8 bytes → decode items A and B → maintain two independent 16-price windows → compute crossings → send exactly 8 bytes back → pass `21_quick_uart_test.py` → pass `22_robust_uart_test.py`.

During the official run, **no team-supplied host software is executed**. All UART parsing, state, computation, and response generation happen on the FPGA.

## Official Submission Channel

**All final hackathon submissions are made through Devpost.** There is no zip upload: each team submits its **GitHub repository URL** and **full final commit SHA**.

Before the deadline, each team should:

1. Build and test the final FPGA design.
2. Push the final source/project files, **including the generated `.fs` file**, to the team's own GitHub repository.
3. Make sure the repository is accessible to judges.
4. Create a final commit and record its **full Git commit SHA**.
5. Complete the Devpost submission with the GitHub repository URL, the full final commit SHA, and the other required project information. The SHA goes in Devpost only, not in your README.
6. Review [SUBMISSION_CHECKLIST.md](SUBMISSION_CHECKLIST.md).
7. Drop off the board and all accessories at Reitz Room 2345 by 11:00 am on Sunday, October 4.

## Start Here

- [Participant Guide (PDF)](participant-resources/GQH_Hardware_Track_Participant_Guide.pdf)
- [Judging & Testing Rules](JUDGING_AND_TESTING.md)
- [Team README Template](TEAM_README_TEMPLATE.md)
- [Recommended Repository Structure](REPOSITORY_STRUCTURE.md)
- [Final Submission Checklist](SUBMISSION_CHECKLIST.md)
- [Participant Testing Resources](participant-resources/README.md)

## Organizer-Provided Files

- [`19_tang_nano_20k.cst`](participant-resources/19_tang_nano_20k.cst) — Tang Nano 20K pin constraints
- [`21_quick_uart_test.py`](participant-resources/testing/21_quick_uart_test.py) — quick UART test ([reference](participant-resources/testing/21_quick_uart_test_REFERENCE.md))
- [`22_robust_uart_test.py`](participant-resources/testing/22_robust_uart_test.py) — robust UART / scoring test ([reference](participant-resources/testing/22_robust_uart_test_REFERENCE.md))

Both scripts need Python 3 and pyserial. Change only the `PORT` setting. See [participant-resources/README.md](participant-resources/README.md).

## Judging Criteria

Each team is judged on **one official run of 100 packets** (indices 0–99). Indices 0–15 are warm-up, leaving **84 scored packets** and **168 scored actions**. The price seed is chosen by the organizers, is the same for every team, and is not published in advance.

| Criterion | Points | How points are awarded |
|---|---:|---|
| Packet correctness | 50 | `50 × correct packets ÷ 84` (index, both item IDs, both actions, and `reserved = 0x0000` all correct) |
| Action correctness | 20 | `20 × correct actions ÷ 168` |
| Latency | 15 | Average round-trip latency vs. the reference design (16.626 ms) on the same judge PC: 15 if ≤ 1.25× (about 20.8 ms), 8 if ≤ 2× (about 33.3 ms), otherwise 0 |
| LUT usage | 15 | `15 × min(1, 542 ÷ your total LUTs)` |
| **Total** | **100** | |

Latency and LUT points are **0** if packet correctness is below **95%**.

> **BL616 warning:** the Tang Nano 20K's USB-serial bridge can drop or corrupt bytes if your design sends response bytes back-to-back. Add idle time or buffering between response bytes. Measured latency includes that delay.

See [JUDGING_AND_TESTING.md](JUDGING_AND_TESTING.md) for the full scoring rules, latency and LUT definitions, the exact moving-average algorithm, the fixed ports, and the judging flow.

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

`reserved` must be `0x0000`. It counts toward correctness: a packet with any other value is incorrect.

### Fixed IDs

```text
ITEM_A = 0x11 = 00010001
ITEM_B = 0x22 = 00100010

NONE = 0x00 = 00000000
SELL = 0x01 = 00000001
BUY  = 0x02 = 00000010
```

Both packets are **64 bits / 8 bytes**. UART is **115200 baud, 8N1, LSB first**. Multi-byte fields are transmitted **big-endian**.

The tester may place either item in either slot on any packet. Route by item ID only, and mirror the request's slot order in the response.

Do not modify the packet layout, field widths, byte order, item IDs, action IDs, or packet length.

## Fixed Top-Level Ports

Port names must match the organizer-supplied `19_tang_nano_20k.cst` exactly and **may not be renamed**. The top-level entity/module name is your choice.

| Port | Pin | Direction | Notes |
|---|---:|---|---|
| `sys_clk` | 4 | in | 27 MHz on-board clock |
| `reset_btn` | 87 | in | Pull-down, optional |
| `uart_rx_i` | 70 | in | BL616 → FPGA |
| `uart_tx_o` | 69 | out | FPGA → BL616 |
| `led0_n` | 15 | out | Active low, optional |
| `led1_n` | 16 | out | Active low, optional |

Participants **do not create their own pin-constraint file** and should not recreate pin assignments in FloorPlanner. Add the supplied `.cst` to the Gowin project as the physical constraint file.

## What the Team Repository Should Contain

At minimum:

- Final HDL source files used by the FPGA design
- The organizer-supplied Tang Nano 20K `.cst` file
- Gowin project/build files needed to reproduce the design
- **The generated `.fs` programming file**, built from that exact source — **required**; judges program the board from it
- A complete `README.md` that includes:
  - Top-level entity/module name
  - Gowin EDA version used
  - Total LUT count from the Gowin synthesis report
  - External libraries, IP cores, starter code, or other pre-existing resources used
  - Known limitations or incomplete features
- Any host-side test or demo code (for local testing/demo only; **not executed during judging**)
- Testbench/simulation files, if used
- Supporting data/configuration files needed to reproduce the project

See [TEAM_README_TEMPLATE.md](TEAM_README_TEMPLATE.md) and [REPOSITORY_STRUCTURE.md](REPOSITORY_STRUCTURE.md).

## Final Commit SHA

Once the final project is ready:

```bash
git status
git add .
git commit -m "Final hackathon submission"
git push
git rev-parse HEAD
```

Save the entire SHA printed by the final command. This identifies the exact version intended for judging. Do not rely only on the repository URL, because the repository can continue changing after the deadline.

Enter the SHA on Devpost. If Devpost has no dedicated field for it, paste it into your Devpost project description, for example:

```text
Final GitHub Submission
Repository: https://github.com/team-name/project-name
Final Commit SHA: 7fe929310cd84d0e1f1d6c1234567890abcdef12
```

Do not put the SHA in your README: a commit cannot contain its own SHA.

## Repository Access

Teams are responsible for ensuring judges can access the submitted repository.

- **Public repository:** verify that the URL opens when you are logged out of GitHub.
- **Private repository:** follow the organizer-provided judging-access instructions and grant access before the deadline. **[TODO: judging-access instructions for private repositories are not yet published.]**

Never commit passwords, API keys, access tokens, private keys, or other secrets.

---

For event-specific questions, use the GQH Hardware Track Discord ([discord.gg/9qPMtN4UB](https://discord.gg/9qPMtN4UB)). For pickup or drop-off questions you can also email IoTStudentsClub@ece.ufl.edu.
