# Quick UART Test Reference

## Purpose

`21_quick_uart_test.py` is the **basic UART communication and packet-format test** for the FPGA Trade Signal Hackathon.

Use this test before running the full scoring test. Its purpose is to verify that:

- The PC can transmit packets to the FPGA over UART.
- The FPGA receives the complete input packet.
- The FPGA returns exactly one correctly formatted output packet.
- The FPGA preserves the transaction index.
- The FPGA preserves and correctly routes the item IDs.
- The FPGA returns valid action codes.
- The complete request/response transaction works at **115200 baud**.

This test is intentionally small and easy to inspect. Passing it is a strong indication that the UART and packet-handling portions of your design are working.

---

# IMPORTANT: THE PACKET PROTOCOL IS FIXED

> **DO NOT MODIFY THE PACKET FORMAT, FIELD ORDER, FIELD WIDTHS, ITEM CODES, OR ACTION CODES.**

Your FPGA design must adapt to the testing protocol. The testing protocol will **not** be adapted to your FPGA design.

Do **not**:

- Rearrange packet fields.
- Add fields.
- Remove fields.
- Change field widths.
- Change endianness.
- Change the numeric/binary encodings of the item IDs.
- Change the numeric/binary encodings of the actions.
- Return the items in a different order from the received packet.
- Substitute ASCII text for the binary packet.
- Add newline, comma, delimiter, header, or footer bytes.
- Return fewer or more than 8 bytes.

Even if a different encoding seems easier for your design, **use the protocol exactly as specified here**.

---

# UART Settings

The test uses:

| Setting | Value |
|---|---:|
| Baud rate | `115200` |
| Data bits | `8` |
| Packet transport | Raw binary bytes |
| PC response timeout | `1.0 s` |

The Python script's `PORT` variable must be changed to the COM port assigned to the Tang Nano board.

Example:

```python
PORT = "COM6"
BAUD = 115200
```

The UART connection transports **raw binary data**. It is not sending strings such as `"BUY"` or `"ITEM_A"`.

---

# Fixed Item Codes

There are two independent trade items.

| Item | Hex | Binary |
|---|---:|---:|
| Item A | `0x11` | `00010001` |
| Item B | `0x22` | `00100010` |

These values are **identifiers**, not prices.

> **Do not change these codes.**
>
> Your FPGA should determine which internal price history/state to update by examining the received item ID.

For example:

```text
00010001 -> Item A
00100010 -> Item B
```

---

# Fixed Action Codes

Each returned action occupies exactly **8 bits**.

| Action | Hex | Binary |
|---|---:|---:|
| NONE / initial no-action state | `0x00` | `00000000` |
| SELL | `0x01` | `00000001` |
| BUY | `0x02` | `00000010` |

> **Do not change or remap these values.**

For example, BUY must be returned as:

```text
00000010
```

not:

```text
00000001
```

and not an ASCII character such as:

```text
"B"
```

The moving-average algorithm may also **hold the previous BUY or SELL action when no new crossing occurs**. "Hold" describes algorithm behavior; it does **not** introduce a new packet code. The returned action byte remains the previously held `BUY` (`0x02`) or `SELL` (`0x01`) value. `NONE` (`0x00`) is the initial/no-action value.

---

# PC -> FPGA Input Packet

Every input transaction is exactly:

```text
[index16][item1_8][price1_16][item2_8][price2_16]
```

Total:

```text
16 + 8 + 16 + 8 + 16 = 64 bits = 8 bytes
```

The Python format is:

```python
struct.Struct(">HBHBH")
```

The `>` means the multi-byte fields use **big-endian byte order**.

## Byte Layout

| Byte | Contents |
|---:|---|
| 0 | `index[15:8]` |
| 1 | `index[7:0]` |
| 2 | `item1[7:0]` |
| 3 | `price1[15:8]` |
| 4 | `price1[7:0]` |
| 5 | `item2[7:0]` |
| 6 | `price2[15:8]` |
| 7 | `price2[7:0]` |

Therefore, your FPGA UART receiver should reconstruct the packet as:

```text
63                     48 47      40 39          24 23      16 15           0
+------------------------+----------+--------------+----------+--------------+
|        INDEX[15:0]     | ITEM1[7:0]| PRICE1[15:0]| ITEM2[7:0]| PRICE2[15:0]|
+------------------------+----------+--------------+----------+--------------+
```

The **first byte transmitted belongs to the most-significant portion of the index**.

---

# FPGA -> PC Output Packet

For every valid 8-byte input packet, the FPGA must return exactly one **8-byte output packet**:

```text
[index16][item1_8][action1_8][item2_8][action2_8][reserved16]
```

Total:

```text
16 + 8 + 8 + 8 + 8 + 16 = 64 bits = 8 bytes
```

The Python format is:

```python
struct.Struct(">HBBBBH")
```

## Byte Layout

| Byte | Contents |
|---:|---|
| 0 | `index[15:8]` |
| 1 | `index[7:0]` |
| 2 | `item1[7:0]` |
| 3 | `action1[7:0]` |
| 4 | `item2[7:0]` |
| 5 | `action2[7:0]` |
| 6 | `reserved[15:8]` |
| 7 | `reserved[7:0]` |

Recommended reserved value:

```text
reserved = 0x0000
```

The packet should therefore be assembled as:

```text
63                     48 47      40 39      32 31      24 23      16 15           0
+------------------------+----------+----------+----------+----------+--------------+
|        INDEX[15:0]     | ITEM1    | ACTION1  | ITEM2    | ACTION2  | RESERVED     |
+------------------------+----------+----------+----------+----------+--------------+
```

---

# Preserve the Received Item Order

This is extremely important.

The test intentionally changes the ordering of Item A and Item B after the warm-up period.

One transaction may contain:

```text
item1 = ITEM_A
item2 = ITEM_B
```

while another may contain:

```text
item1 = ITEM_B
item2 = ITEM_A
```

Your FPGA must use the **item ID**, rather than the packet position, to determine which item's moving-average state is being updated.

If the input packet contains:

```text
[index][ITEM_B][price_B][ITEM_A][price_A]
```

then the response must be:

```text
[index][ITEM_B][action_B][ITEM_A][action_A][reserved]
```

Do **not** force Item A into output position 1 and Item B into output position 2.

---

# What the Quick Test Sends

The script contains short deterministic price sequences for both items.

The first 16 samples fill the moving-average history. After that, the prices intentionally move enough to exercise the trade-signal logic.

The test also swaps the packet order on alternating transactions after index 15. This tests whether your FPGA routes information by the **item ID** instead of assuming that packet position 1 always means Item A.

---

# 16-Sample Warm-Up

The algorithm uses a **16-sample moving-average window**.

Therefore:

```text
indices 0 through 15 = WARMUP
```

These first 16 samples are used to populate the moving-average history.

Participants should not assume that these samples represent normal scored trade decisions.

A good hardware design should initialize/reset its state cleanly so that the first packet begins a new test sequence.

---

# Stop-and-Wait Communication

The PC uses a simple stop-and-wait protocol:

```text
PC sends packet N
        |
        v
FPGA receives all 8 bytes
        |
        v
FPGA processes packet N
        |
        v
FPGA returns all 8 bytes
        |
        v
PC receives response N
        |
        v
PC may send packet N+1
```

The Python test does:

```python
ser.write(tx)
rx = ser.read(8)
```

Therefore, your FPGA must return a response for **every input packet**.

Do not wait for another input packet before transmitting the current result.

---

# Latency Measurement

Immediately before sending the input packet, the PC records a timestamp.

It records another timestamp after all 8 output bytes have been received.

Conceptually:

```text
latency = time_after_complete_response - time_before_transmit
```

This is a **round-trip software-observed latency**, so it includes:

- PC UART transmission
- FPGA UART reception
- FPGA processing
- FPGA UART transmission
- PC UART reception
- Host/USB/serial overhead

It is not purely the FPGA algorithm's internal clock-cycle latency.

---

# What You Should See

For each transaction, the terminal prints information similar to:

```text
idx= 16 | item 0x11: BUY  | item 0x22: SELL | reserved=0x0000 | VALID | 1500.0 us
```

During the first 16 transactions it reports:

```text
WARMUP
```

Afterward it reports:

```text
VALID
```

This quick test is primarily intended to verify that communication and packet routing work before using the full scoring test.

---

# Timeout Behavior

The PC expects exactly 8 response bytes.

If it does not receive all 8 bytes within the serial timeout, the script raises a timeout error.

Common causes include:

- Incorrect COM port
- Incorrect baud rate
- Incorrect FPGA clock/baud divider
- UART TX not connected
- FPGA never starting transmission
- FPGA transmitting fewer than 8 bytes
- Packet TX state machine stopping early
- Reset logic holding part of the design in reset

---

# Participant Checklist

Before running the full test, verify:

- UART is configured for `115200` baud.
- FPGA receives exactly 8 bytes per input packet.
- FPGA sends exactly 8 bytes per response.
- Input packet is decoded as `[index, item1, price1, item2, price2]`.
- Output packet is encoded as `[index, item1, action1, item2, action2, reserved]`.
- Multi-byte values are big-endian on the wire.
- `ITEM_A = 0x11`.
- `ITEM_B = 0x22`.
- `NONE = 0x00`.
- `SELL = 0x01`.
- `BUY = 0x02`.
- No new "HOLD" packet code is invented; holding means retaining the previous BUY/SELL action.
- Item IDs determine routing.
- Returned item order matches the received item order.
- The returned index matches the received index.
- Reserved bits are preferably `0x0000`.
- The design starts from a known state at the beginning of the test.

---

# Final Warning

## DO NOT EDIT THE PROTOCOL TO MATCH YOUR DESIGN

Your implementation is being tested against a common interface.

The following are part of the competition specification and are **not participant-configurable**:

```text
INPUT:
[index16][item1_8][price1_16][item2_8][price2_16]

OUTPUT:
[index16][item1_8][action1_8][item2_8][action2_8][reserved16]

ITEM_A = 0x11
ITEM_B = 0x22

NONE = 0x00
SELL = 0x01
BUY  = 0x02

UART = 115200 baud
PACKET SIZE = 8 bytes in each direction
BYTE ORDER = big-endian for multi-byte fields
```

**Implement your FPGA around this interface. Do not change the interface around your FPGA.**