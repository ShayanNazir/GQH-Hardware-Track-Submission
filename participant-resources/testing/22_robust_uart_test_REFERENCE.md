# Robust UART Trade-Signal Test Reference

## Purpose

`22_robust_uart_test.py` is the **full functional and scoring-style test** for the FPGA Trade Signal Hackathon.

Unlike the quick UART test, this script:

* Generates **100 test packets** (indices 0–99), the same length as the official judging run.
* Uses a fixed **practice seed**. The official judging seed is different.
* Tests two independent items.
* Calculates the expected results in software **before transmission**.
* Exercises the 16-sample moving-average trade algorithm.
* After warm-up, randomly places Item A and Item B in either slot on every packet to test ID-based routing.
* Sends only one transaction at a time (stop-and-wait).
* Checks the returned index, both item IDs, both actions, and `reserved = 0x0000`, matching the official packet-correctness definition.
* Detects incomplete UART responses/timeouts (1.0 s per packet).
* Measures round-trip latency.
* Scores against the fixed official totals: **84 scored packets** and **168 scored actions**.
* Produces a CSV containing the detailed results.
* Produces a TXT summary containing final correctness and latency statistics.

Participants should first establish basic communication using `21_quick_uart_test.py`, then use this script for full verification.

The full rules are in the [participant guide](../GQH_Hardware_Track_Participant_Guide.pdf) and [JUDGING_AND_TESTING.md](../../JUDGING_AND_TESTING.md). If this page disagrees with the guide, the guide wins.

> **[TODO]** `22_robust_uart_test.py` is not yet in this repository. This page describes the participant version as it must behave: `PACKET_COUNT = 100`, a fixed practice seed (never the official judging seed), seeded random slot placement after warm-up, fixed 84/168 scoring totals, `reserved != 0x0000` treated as an incorrect packet (with a `RESERVED` status in the CSV), 1.0 s timeout, stop-and-wait, and CSV/summary output for 100 packets.

---

# WARNING: THE BL616 USB-SERIAL BRIDGE

> The Tang Nano 20K's onboard BL616 USB-serial bridge can **drop or corrupt bytes**, causing timeouts, if your FPGA sends response bytes **back-to-back with no idle time**. A functionally correct design can still fail this way.
>
> Your design **must** add idle time or buffering between response bytes. The measured latency includes that added delay, so there is a trade-off: more idle time is safer but slower. Check the CSV to see which packets failed.

---

# CRITICAL COMPETITION RULE: DO NOT CHANGE THE PACKET PROTOCOL

> **The packet format and encoded values are part of the competition interface. They must not be changed.**

Participants must **NOT** change:

* Input field order.
* Output field order.
* Field widths.
* Packet length.
* Endianness.
* Item identifiers.
* Action identifiers.
* Meaning of the action identifiers.
* Placement of Item 1 and Item 2 fields.
* The requirement to return the same transaction index.
* The requirement to preserve the received item order.

Do not edit the testing script to make an incompatible FPGA implementation pass.

Your FPGA must conform to the protocol below.

---

# Test Configuration

The full test uses:

```text
UART:           115200 baud, 8N1, LSB first
Packet count:   100 (indices 0-99)
Warm-up:        indices 0-15
Scored:         84 packets, 168 actions
Window size:    16
Price range:    0 through 100
Timeout:        1.0 s per packet
```

The price range above is this script's setting.

The script needs **Python 3** and **pyserial** (`pip install pyserial`).

## Seed

The script uses a fixed **practice seed**, so the generated price sequence is repeatable: running the same unmodified test produces the same test vectors and the same slot placements.

The official judging seed is **different**. It is chosen by the organizers, is the same for every team, and is not published. Do not hardcode price patterns.

## What You May Change

The participant may need to change:

```python
PORT = "COM6"
```

to the COM port assigned to their Tang Nano board.

Changing the COM port is expected. It is the **only** setting you may change.

Changing the packet protocol or scoring logic is **not** allowed.

---

# Fixed Item IDs

The test defines:

| Item | Hex | Binary |
|---|---:|---:|
| Item A | `0x11` | `00010001` |
| Item B | `0x22` | `00100010` |

These IDs allow the FPGA to determine which independent moving-average state belongs to each received price.

> **Do not change these values.**

The packet position is not a substitute for the item ID.

Your logic should effectively recognize:

```text
if item_id == 0x11:
    use Item A state/history

if item_id == 0x22:
    use Item B state/history
```

---

# Fixed Action Codes

Every returned action is exactly 8 bits.

| Meaning | Hex | Binary |
|---|---:|---:|
| NONE / initial no-action state | `0x00` | `00000000` |
| SELL | `0x01` | `00000001` |
| BUY | `0x02` | `00000010` |

These encodings are fixed.

> **Do not reverse BUY and SELL. Do not invent a different encoding.**

In particular:

```text
SELL = 00000001
BUY  = 00000010
```

The algorithm also has **hold behavior**: if no new crossing occurs, the previous action is retained. "HOLD" is not a fourth output encoding. If the previous action was BUY, holding means the returned action remains `0x02`; if the previous action was SELL, it remains `0x01`. Before an item's first crossing, its action is `NONE = 0x00`.

---

# PC -> FPGA Packet

Every PC-to-FPGA packet contains exactly 64 bits:

```text
[index16][item1_8][price1_16][item2_8][price2_16]
```

Python representation:

```python
INPUT_STRUCT = struct.Struct(">HBHBH")
```

where:

```text
H = unsigned 16-bit value
B = unsigned 8-bit value
> = big-endian
```

## Exact Byte Order

| Byte | Field |
|---:|---|
| 0 | `index[15:8]` |
| 1 | `index[7:0]` |
| 2 | `item1[7:0]` |
| 3 | `price1[15:8]` |
| 4 | `price1[7:0]` |
| 5 | `item2[7:0]` |
| 6 | `price2[15:8]` |
| 7 | `price2[7:0]` |

Hardware-oriented view:

```text
63                     48 47      40 39          24 23      16 15           0
+------------------------+----------+--------------+----------+--------------+
|        INDEX[15:0]     | ITEM1    | PRICE1[15:0] | ITEM2    | PRICE2[15:0] |
+------------------------+----------+--------------+----------+--------------+
```

The packet contains **two prices in every transaction**, one for each identified item.

---

# FPGA -> PC Packet

For every complete input packet, the FPGA must return exactly:

```text
[index16][item1_8][action1_8][item2_8][action2_8][reserved16]
```

Python representation:

```python
OUTPUT_STRUCT = struct.Struct(">HBBBBH")
```

Total:

```text
64 bits = 8 bytes
```

## Exact Byte Order

| Byte | Field |
|---:|---|
| 0 | `index[15:8]` |
| 1 | `index[7:0]` |
| 2 | `item1[7:0]` |
| 3 | `action1[7:0]` |
| 4 | `item2[7:0]` |
| 5 | `action2[7:0]` |
| 6 | `reserved[15:8]` |
| 7 | `reserved[7:0]` |

Hardware-oriented view:

```text
63                     48 47      40 39      32 31      24 23      16 15           0
+------------------------+----------+----------+----------+----------+--------------+
|        INDEX[15:0]     | ITEM1    | ACTION1  | ITEM2    | ACTION2  | RESERVED     |
+------------------------+----------+----------+----------+----------+--------------+
```

The reserved field **must** be:

```text
reserved = 0x0000
```

`reserved` counts toward packet correctness. In this test and in the official run, a packet with any other value is incorrect.

---

# Example Packet

This is the worked example from the participant guide.

Suppose the PC sends:

```text
index  = 16
item1  = Item A = 0x11
price1 = 80
item2  = Item B = 0x22
price2 = 200
```

The logical packet is:

```text
[0x0010][0x11][0x0050][0x22][0x00C8]
```

The 8 UART bytes are:

```text
00 10 11 00 50 22 00 C8
```

If the FPGA decides:

```text
Item A -> SELL
Item B -> BUY
```

then the response should logically be:

```text
[0x0010][0x11][0x01][0x22][0x02][0x0000]
```

and the actual 8 transmitted bytes are:

```text
00 10 11 01 22 02 00 00
```

---

# Why Item IDs Matter

After the 16-packet warm-up, this test uses its seeded random generator to choose, on every packet, which slot each item occupies. The official tester may place either item in either slot on **any** packet.

For one index the PC may send:

```text
[index][ITEM_A][price_A][ITEM_B][price_B]
```

and for another:

```text
[index][ITEM_B][price_B][ITEM_A][price_A]
```

This checks whether your FPGA routes by **ID** rather than assuming:

```text
slot 1 = Item A
slot 2 = Item B
```

That assumption is incorrect. Route exclusively by item ID, never by packet index or slot position.

If the PC sends:

```text
[index][ITEM_B][price_B][ITEM_A][price_A]
```

the FPGA must return:

```text
[index][ITEM_B][action_B][ITEM_A][action_A][reserved]
```

The response packet follows the **same item ordering as that input transaction**.

---

# Software Reference Model

Before UART transmission begins, the script generates all 100 prices for Item A and all 100 prices for Item B.

It then runs those prices through two independent software reference models:

```text
Item A -> MovingAverageReference A
Item B -> MovingAverageReference B
```

The expected FPGA actions are therefore calculated **before the hardware test begins**.

The FPGA's answers are compared against this precomputed reference.

The exact algorithm is also in [JUDGING_AND_TESTING.md](../../JUDGING_AND_TESTING.md#exact-16-sample-moving-average-algorithm).

---

# 16-Sample Moving Average

Each item maintains its own 16-sample window.

## Reset at Index 0

Index 0 starts a new session. On receipt of index 0, clear all previous state for both items (every window, sum, previous price, and last action) **first**, then process index 0's two prices as the first samples of the new window. The board is not reset or reprogrammed between runs, so your design must do this by itself.

## Warm-Up

During the first 16 samples:

```text
index 0 through index 15
```

for each item, the reference:

```text
adds the price to the window and running sum
stores the price as the item's previous price
does not evaluate a crossing
expects NONE (0x00) for the action
```

So at index 16, the previous price is the price from index 15.

These packets are treated as:

```text
IGNORED_WARMUP
```

and are not included in correctness scoring. Their responses must still follow the protocol and arrive in time.

## Averages

From index 16 onward, the reference computes:

```text
old_average = old_sum >> 4
```

Since the window contains 16 values:

```text
running_sum / 16 = running_sum >> 4
```

for unsigned prices. The fraction is discarded, not rounded.

For the incoming price:

```text
new_sum     = old_sum - oldest_price + current_price
new_average = new_sum >> 4
```

This means the reference uses **floor division by 16**, not floating-point averaging.

---

# BUY / SELL Decision Logic

From index 16 onward, the software checks for crossings.

## BUY

A BUY occurs when:

```text
previous_price <= old_average
AND
current_price > new_average
```

Then:

```text
action = BUY = 0x02
```

## SELL

A SELL occurs when:

```text
previous_price >= old_average
AND
current_price < new_average
```

Then:

```text
action = SELL = 0x01
```

## No New Crossing / Hold

If neither crossing occurs:

```text
action remains unchanged
```

This is important.

The algorithm does **not** automatically return `NONE` every time there is no new crossing. It **holds the previous action**.

For example:

```text
previous stored action = BUY (0x02)
no new crossing
returned action = BUY (0x02)
```

Again, **do not create a new HOLD code**. Hold is state behavior, not a separate packet encoding.

## After Every Update

```text
previous_price = current_price
```

---

# Two Independent Histories

Item A and Item B must have independent algorithm state.

Conceptually:

```text
ITEM A:
    16-price history
    running sum (20 bits)
    previous price
    current held action

ITEM B:
    16-price history
    running sum (20 bits)
    previous price
    current held action
```

Do not mix the histories.

The item ID determines which state receives a particular price.

---

# Stop-and-Wait UART Protocol

The full test intentionally sends only one transaction at a time.

```text
Send packet N
      |
      v
Wait for exactly 8 response bytes
      |
      v
Check response N
      |
      v
Send packet N+1
```

In the Python script this is fundamentally:

```python
ser.write(tx)
rx = ser.read(8)
```

The next packet is **not sent until the current read completes**.

This makes the protocol simple for FPGA implementations because multiple outstanding requests do not need to be tracked.

---

# What Counts as a Correct Packet

After warm-up, the script checks six things:

```text
1. returned index    == transmitted index
2. returned item1    == transmitted item1
3. returned action1  == software expected action1
4. returned item2    == transmitted item2
5. returned action2  == software expected action2
6. returned reserved == 0x0000
```

A packet is counted as correct only if **all six checks pass**. This is the same packet-correctness definition the official run uses.

Therefore, returning the correct BUY/SELL decisions with the wrong item IDs, the wrong index, or a nonzero reserved field still produces an incorrect packet.

---

# Action Correctness vs Packet Correctness

The test reports two correctness measurements, both against **fixed totals**. They are not divided by the number of packets received, so a run that stops early scores zero for every packet it did not receive.

## Action Correctness

Each returned action is checked independently.

There are two scored actions per post-warm-up packet:

```text
action1
action2
```

```text
action correctness = correct actions / 168
```

This indicates how often individual trade decisions were correct.

## Packet Correctness

A packet is correct only when the complete required response matches:

```text
index
item1
action1
item2
action2
reserved
```

```text
packet correctness = correct packets / 84
```

Packet correctness is therefore stricter.

---

# Timeout / Partial Packet Handling

The PC requests exactly 8 bytes:

```python
rx = ser.read(8)
```

If fewer than 8 bytes arrive within **1.0 s**, the test records a `TIMEOUT`, counts that packet as incorrect, and stops. Packets that were never sent score zero. This matches the official run.

The test intentionally stops after an incomplete packet because continuing could cause subsequent UART bytes to become misaligned with packet boundaries.

Therefore:

> Your FPGA should always transmit exactly one complete 8-byte response for each complete 8-byte request.

If your logic is correct but you still see timeouts, read the BL616 warning at the top of this page.

---

# Latency Measurement

For each transaction:

```text
t0 = immediately before PC transmission
t1 = immediately after the complete 8-byte FPGA response
```

Then:

```text
latency_us = (t1 - t0) / 1000
```

The final summary reports the average latency for successfully received packets.

This is a **round-trip system measurement**. It includes UART transfer in both directions, FPGA processing, any idle time your design adds between response bytes, and serial/USB/host overhead. At 115200 baud (8N1), the 16 bytes of one transaction need only about 1.39 ms of wire time; most of the roughly 16.6 ms reference latency is BL616, USB, operating-system, and serial-buffering overhead.

---

# CSV Output

The test creates:

```text
trade_results_100.csv
```

The CSV records fields including:

```text
index

tx_item1
tx_price1
tx_item2
tx_price2

expected_action1
expected_action2

rx_index
rx_item1
rx_action1
rx_item2
rx_action2
rx_reserved

action1_correct
action2_correct
packet_correct

status
latency_us
```

This file is useful for debugging because it shows exactly which portion of a failed transaction did not match. Keep the CSV from every run.

Possible status information can identify failures such as:

```text
INDEX
ITEM1
ACTION1
ITEM2
ACTION2
RESERVED
TIMEOUT
```

Any of these statuses marks the packet incorrect. `RESERVED` means the response's reserved field was not `0x0000`.

---

# Summary TXT Output

The test also creates:

```text
trade_summary_100.txt
```

It reports:

* Requested packet count (100).
* Successfully received packet count.
* Number of ignored warm-up packets (16).
* Number of scored packets (84).
* Correct packet count.
* Packet correctness percentage (out of 84).
* Correct individual action count.
* Action correctness percentage (out of 168).
* Timeout count.
* Average successful round-trip latency.
* UART port.
* UART baud rate.
* Seed used (the practice seed, not the official judging seed).

This provides a compact final test result.

---

# Expected Test Sequence

Conceptually, the test performs:

```text
1. Generate 100 prices for Item A from the practice seed.
2. Generate 100 prices for Item B from the practice seed.

3. Run all prices through the software reference model.
4. Save the expected actions.

5. Open UART at 115200 baud.

6. For index = 0 through 99:

      obtain Item A and Item B prices

      if index >= 16:
          use the seeded RNG to choose which slot each item occupies

      construct exactly 8 input bytes

      start latency timer

      transmit input packet

      wait for exactly 8 output bytes (1.0 s timeout)

      stop latency timer

      if response is incomplete:
          record TIMEOUT (packet incorrect)
          stop test

      decode response

      if index < 16:
          mark as warm-up
      else:
          compare index
          compare item IDs
          compare actions
          check reserved == 0x0000
          score result

      save CSV row

7. Calculate final statistics against 84 packets / 168 actions.
8. Write CSV.
9. Write summary TXT.
10. Print final results.
```

---

# What Participants May Change

Only the serial port:

```python
PORT = "COM6"
```

For example:

```python
PORT = "COM4"
```

depending on the port assigned by Windows (see Device Manager).

---

# What Participants Must NOT Change

Do **not** modify these competition protocol values:

```text
ITEM_A = 0x11
ITEM_B = 0x22

NONE = 0x00
SELL = 0x01
BUY  = 0x02
```

Do not create a separate HOLD encoding. Holding means retaining the previous action value.

Do **not** modify the input packet:

```text
[index16][item1_8][price1_16][item2_8][price2_16]
```

Do **not** modify the output packet:

```text
[index16][item1_8][action1_8][item2_8][action2_8][reserved16]
```

Do **not** reorder fields.

Do **not** force Item A and Item B into fixed packet slots.

Do **not** change the packet size from 8 bytes.

Do **not** return ASCII text.

Do **not** change the byte order.

Do **not** change the expected BUY/SELL algorithm or the scoring logic in the test script in order to make an incompatible FPGA result appear correct.

---

# Common FPGA Implementation Mistakes

## 1. Reversing the action codes

Wrong:

```text
BUY  = 01
SELL = 10
```

Required:

```text
SELL = 00000001
BUY  = 00000010
```

Remember that each action field is **8 bits**, even though only small numeric values are currently used.

---

## 2. Treating slot 1 as permanently Item A

Wrong:

```text
price1 always updates A
price2 always updates B
```

Required:

```text
item1 ID determines where price1 goes
item2 ID determines where price2 goes
```

---

## 3. Returning items in a fixed A/B order

Wrong if the request was B/A:

```text
response = A/actionA, B/actionB
```

Required:

```text
request  = B/priceB, A/priceA
response = B/actionB, A/actionA
```

---

## 4. Using floating-point or rounded averaging

The reference uses:

```text
average = running_sum >> 4
```

Match this integer (floor) behavior.

---

## 5. Returning NONE whenever there is no crossing

The reference holds the previous action.

Wrong:

```text
no crossing -> NONE
```

Required:

```text
no crossing -> previous action remains unchanged
```

---

## 6. Not updating the previous price during warm-up

Wrong:

```text
previous_price is first set at index 16
```

Required:

```text
previous_price is updated on every warm-up packet,
so at index 16 it holds the price from index 15
```

---

## 7. Responding before the entire request is decoded

Wait until the complete 8-byte input packet has been received and reconstructed before processing it. Never send unsolicited bytes.

---

## 8. Sending an incorrect number of response bytes

The PC expects:

```text
8 bytes exactly
```

An incomplete response ends the test.

---

## 9. Sending response bytes back-to-back

The BL616 bridge can drop or corrupt bytes when there is no idle time between response bytes. Add idle time or buffering between response bytes.

---

## 10. Leaving reserved nonzero

```text
reserved = 0x0000
```

Any other value makes the packet incorrect, in this test and in the official run.

---

# Final Protocol Cheat Sheet

```text
UART
----
Baud = 115200, 8N1, LSB first

ITEM IDs
--------
ITEM_A = 0x11 = 00010001
ITEM_B = 0x22 = 00100010

ACTION IDs
----------
NONE = 0x00 = 00000000
SELL = 0x01 = 00000001
BUY  = 0x02 = 00000010

No separate HOLD code:
no crossing -> retain previous action

PC -> FPGA
----------
64 bits / 8 bytes

[index16][item1_8][price1_16][item2_8][price2_16]

Bytes:
0 index[15:8]
1 index[7:0]
2 item1
3 price1[15:8]
4 price1[7:0]
5 item2
6 price2[15:8]
7 price2[7:0]

FPGA -> PC
----------
64 bits / 8 bytes

[index16][item1_8][action1_8][item2_8][action2_8][reserved16]

Bytes:
0 index[15:8]
1 index[7:0]
2 item1
3 action1
4 item2
5 action2
6 reserved[15:8] = 0x00
7 reserved[7:0]  = 0x00

MULTI-BYTE ORDER
----------------
Big-endian

RESET
-----
Index 0 = clear all state, then process index 0 prices

MOVING AVERAGE
--------------
Window = 16 samples
Indices 0-15 = warm-up / not scored
  (add to window and sum, update previous price, respond NONE)
old_average = old_sum >> 4
new_sum     = old_sum - oldest_price + current_price
new_average = new_sum >> 4

BUY
---
previous_price <= old_average
AND
current_price > new_average

SELL
----
previous_price >= old_average
AND
current_price < new_average

OTHERWISE
---------
retain previous action

THEN
----
previous_price = current_price

TRANSPORT
---------
stop-and-wait:
send one 8-byte request
wait for one 8-byte response (1.0 s timeout)
then send next request
```

---

# Competition Rule Summary

The testing scripts define the external hardware/software interface.

**Participants implement the algorithm and hardware architecture. Participants do not redefine the packet protocol.**

If your FPGA produces a different packet format, item encoding, action encoding, field ordering, or byte ordering, the official tester will interpret those bytes according to the specification above and the result will be incorrect.

Design your FPGA to match the tester, not the tester to match your FPGA.
