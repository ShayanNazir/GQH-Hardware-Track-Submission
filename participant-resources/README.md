# Participant Resources

This folder is the distribution point for organizer-provided Hardware Track files.

## Participant Guide

- [GQH Hardware Track Participant Guide (PDF)](GQH_Hardware_Track_Participant_Guide.pdf) — pickup/drop-off, Tang Nano 20K setup and programming, judging, scoring, and submission. If any other file in this repository disagrees with the guide, the guide wins.

## Files Provided by the Organizers

| File | Purpose |
|---|---|
| `constraints/19_tang_nano_20k.cst` | Board pin mappings. Add it to your Gowin project as the physical constraint file. **[TODO: not yet in this repository]** |
| `testing/21_quick_uart_test.py` | Quick sanity test: basic communication and packet format. Run this first. **[TODO: not yet in this repository]** |
| `testing/22_robust_uart_test.py` | Scoring-style test with a software reference model. Run this second. **[TODO: not yet in this repository]** |

```text
participant-resources/
├── GQH_Hardware_Track_Participant_Guide.pdf
├── constraints/
│   └── 19_tang_nano_20k.cst
└── testing/
    ├── 21_quick_uart_test.py
    ├── 21_quick_uart_test_REFERENCE.md
    ├── 22_robust_uart_test.py
    └── 22_robust_uart_test_REFERENCE.md
```

## Testing References

- [Quick UART Test Reference](testing/21_quick_uart_test_REFERENCE.md)
- [Robust UART / Scoring Test Reference](testing/22_robust_uart_test_REFERENCE.md)

## Running the Scripts

Both scripts need **Python 3** and **pyserial**:

```bash
pip install pyserial
```

Change **only** the `PORT` setting to your board's COM port (find it in Windows Device Manager). Only one program can open the port at a time, so close Gowin Programmer, serial terminals, and other scripts first.

**Never** change the packet protocol, item/action encodings, or scoring logic to make a design pass.

The robust test saves a CSV of every packet. Keep it: it is the best debugging tool you have.

## Constraint File

The `.cst` file is provided because board-specific pin discovery is not the intended challenge. Top-level port names must match it exactly and may not be renamed. The UART modules, packet logic, and testbenches are yours to write.
