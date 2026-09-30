# Participant Resources

This folder is the distribution point for organizer-provided Hardware Track files.

## Testing References

- [Quick UART Test Reference](testing/21_quick_uart_test_REFERENCE.md)
- [Robust UART / Scoring Test Reference](testing/22_robust_uart_test_REFERENCE.md)

## Files to Be Added by Organizers

When final versions are provided, this repository should also contain:

```text
testing/
├── 21_quick_uart_test.py
├── 21_quick_uart_test_REFERENCE.md
├── 22_robust_uart_test.py
└── 22_robust_uart_test_REFERENCE.md

constraints/
└── 19_tang_nano_20k.cst
```

Participants should download and use the organizer-provided versions of these files.

Do not change the fixed packet protocol, item/action encodings, or scoring logic in the official tester. The local COM/serial port may be changed as needed.
