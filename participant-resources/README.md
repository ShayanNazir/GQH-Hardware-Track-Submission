# Participant Resources

This folder is the distribution point for organizer-provided Hardware Track files.

## Files Provided by the Organizers

| File | Purpose |
|---|---|
| [`GQH_Hardware_Track_Participant_Guide.pdf`](GQH_Hardware_Track_Participant_Guide.pdf) | Participant guide: pickup/drop-off, Tang Nano 20K setup and programming, judging, scoring, and submission. If any other file in this repository disagrees with the guide, the guide wins. |
| [`19_tang_nano_20k.cst`](19_tang_nano_20k.cst) | Board pin mappings. Add it to your Gowin project as the physical constraint file. |
| [`testing/21_quick_uart_test.py`](testing/21_quick_uart_test.py) | Quick sanity test: basic communication and packet format. Run this first. |
| [`testing/22_robust_uart_test.py`](testing/22_robust_uart_test.py) | Scoring-style test with a software reference model. Run this second. |
| [`testing/21_quick_uart_test_REFERENCE.md`](testing/21_quick_uart_test_REFERENCE.md) | What the quick test sends, checks, and prints. |
| [`testing/22_robust_uart_test_REFERENCE.md`](testing/22_robust_uart_test_REFERENCE.md) | What the robust test sends, checks, scores, and writes. |

```text
participant-resources/
├── GQH_Hardware_Track_Participant_Guide.pdf
├── 19_tang_nano_20k.cst
└── testing/
    ├── 21_quick_uart_test.py
    ├── 21_quick_uart_test_REFERENCE.md
    ├── 22_robust_uart_test.py
    └── 22_robust_uart_test_REFERENCE.md
```

## Running the Scripts

Both scripts need **Python 3** and **pyserial**:

```bash
pip install pyserial
```

Change **only** the `PORT` setting at the top of each script to your board's COM port (find it in Windows Device Manager). Only one program can open the port at a time, so close Gowin Programmer, serial terminals, and other scripts first.

**Never** change the packet protocol, item/action encodings, or scoring logic to make a design pass.

- The quick test prints `OK` or `MISMATCH` for each scored packet and ends with `PASS` or a mismatch count.
- The robust test writes `trade_results_100.csv` (every packet) and `trade_summary_100.txt`, including estimated correctness points out of 70. Keep the CSV: it is the best debugging tool you have.

The robust test uses the **practice seed** `0x57214720`. The official judging seed is different: it is chosen by the organizers, is the same for every team, and is not published.

## Constraint File

The `.cst` file is provided because board-specific pin discovery is not the intended challenge. Top-level port names must match it exactly and may not be renamed:

| Port | Pin | Direction | Notes |
|---|---:|---|---|
| `sys_clk` | 4 | in | 27 MHz on-board clock |
| `reset_btn` | 87 | in | Pull-down, optional |
| `uart_rx_i` | 70 | in | BL616 → FPGA |
| `uart_tx_o` | 69 | out | FPGA → BL616 |
| `led0_n` | 15 | out | Active low, optional |
| `led1_n` | 16 | out | Active low, optional |

The UART modules, packet logic, and testbenches are yours to write.
