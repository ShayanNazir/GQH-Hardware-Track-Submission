# Robust UART Trade-Signal Test Reference

## Purpose

`22\_robust\_uart\_test.py` is the **full functional and scoring test** for the FPGA Trade Signal Hackathon.

Unlike the quick UART test, this script:

* Generates 100 deterministic test packets (Real judging test will generate 1000 with random seed).
* Tests two independent items.
* Calculates the expected results in software **before transmission**.
* Exercises the 16-sample moving-average trade algorithm.
* Changes Item A / Item B packet positions to test ID-based routing.
* Sends only one transaction at a time.
* Checks the returned index.
* Checks the returned item IDs.
* Checks both returned actions.
* Detects incomplete UART responses/timeouts.
* Measures round-trip latency.
* Produces a CSV containing the detailed results.
* Produces a TXT summary containing final correctness and latency statistics.

Participants should first establish basic communication using `21\_quick\_uart\_test.py`, then use this script for full verification.

\---

# CRITICAL COMPETITION RULE: DO NOT CHANGE THE PACKET PROTOCOL

> \*\*The packet format and encoded values are part of the competition interface. They must not be changed.\*\*

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

\---

# Test Configuration

The full test uses:

```text
UART baud rate: 115200
Packet count:   100
Window size:    16
Price range:    0 through 100
```

The test uses the deterministic random seed:

```text
0x57214720
```

This means the generated price sequence is repeatable. Running the same unmodified test produces the same software-generated test vectors.

The participant may need to change:

```python
PORT = "COM6"
```

to the COM port assigned to their Tang Nano board.

Changing the COM port is expected.

Changing the packet protocol or scoring logic is **not**.

\---

# Fixed Item IDs

The test defines:

|Item|Hex|Binary|
|-|-:|-:|
|Item A|`0x11`|`00010001`|
|Item B|`0x22`|`00100010`|

These IDs allow the FPGA to determine which independent moving-average state belongs to each received price.

> \*\*Do not change these values.\*\*

The packet position is not a substitute for the item ID.

Your logic should effectively recognize:

```text
if item\_id == 0x11:
    use Item A state/history

if item\_id == 0x22:
    use Item B state/history
```

\---

# Fixed Action Codes

Every returned action is exactly 8 bits.

|Meaning|Hex|Binary|
|-|-:|-:|
|NONE / initial no-action state|`0x00`|`00000000`|
|SELL|`0x01`|`00000001`|
|BUY|`0x02`|`00000010`|

These encodings are fixed.

> \*\*Do not reverse BUY and SELL. Do not invent a different encoding.\*\*

In particular:

```text
SELL = 00000001
BUY  = 00000010
```

The algorithm also has **hold behavior**: if no new crossing occurs, the previous action is retained. "HOLD" is not a fourth output encoding in these scripts. If the previous action was BUY, holding means the returned action remains `0x02`; if the previous action was SELL, it remains `0x01`. Before an action has been established, the state begins as `NONE = 0x00`.

\---

# PC -> FPGA Packet

Every PC-to-FPGA packet contains exactly 64 bits:

```text
\[index16]\[item1\_8]\[price1\_16]\[item2\_8]\[price2\_16]
```

Python representation:

```python
INPUT\_STRUCT = struct.Struct(">HBHBH")
```

where:

```text
H = unsigned 16-bit value
B = unsigned 8-bit value
> = big-endian
```

## Exact Byte Order

|Byte|Field|
|-:|-|
|0|`index\[15:8]`|
|1|`index\[7:0]`|
|2|`item1\[7:0]`|
|3|`price1\[15:8]`|
|4|`price1\[7:0]`|
|5|`item2\[7:0]`|
|6|`price2\[15:8]`|
|7|`price2\[7:0]`|

Hardware-oriented view:

```text
63                     48 47      40 39          24 23      16 15           0
+------------------------+----------+--------------+----------+--------------+
|        INDEX\[15:0]     | ITEM1    | PRICE1\[15:0] | ITEM2    | PRICE2\[15:0] |
+------------------------+----------+--------------+----------+--------------+
```

The packet contains **two prices in every transaction**, one for each identified item.

\---

# FPGA -> PC Packet

For every complete input packet, the FPGA must return exactly:

```text
\[index16]\[item1\_8]\[action1\_8]\[item2\_8]\[action2\_8]\[reserved16]
```

Python representation:

```python
OUTPUT\_STRUCT = struct.Struct(">HBBBBH")
```

Total:

```text
64 bits = 8 bytes
```

## Exact Byte Order

|Byte|Field|
|-:|-|
|0|`index\[15:8]`|
|1|`index\[7:0]`|
|2|`item1\[7:0]`|
|3|`action1\[7:0]`|
|4|`item2\[7:0]`|
|5|`action2\[7:0]`|
|6|`reserved\[15:8]`|
|7|`reserved\[7:0]`|

Hardware-oriented view:

```text
63                     48 47      40 39      32 31      24 23      16 15           0
+------------------------+----------+----------+----------+----------+--------------+
|        INDEX\[15:0]     | ITEM1    | ACTION1  | ITEM2    | ACTION2  | RESERVED     |
+------------------------+----------+----------+----------+----------+--------------+
```

Use:

```text
reserved = 0x0000
```

unless the competition specification explicitly assigns another purpose to these bits.

\---

# Example Packet

Suppose the PC sends:

```text
index  = 16
item1  = Item A = 0x11
price1 = 80
item2  = Item B = 0x22
price2 = 60
```

The logical packet is:

```text
\[0x0010]\[0x11]\[0x0050]\[0x22]\[0x003C]
```

The 8 UART bytes are:

```text
00 10 11 00 50 22 00 3C
```

If the FPGA decides:

```text
Item A -> BUY
Item B -> SELL
```

then the response should logically be:

```text
\[0x0010]\[0x11]\[0x02]\[0x22]\[0x01]\[0x0000]
```

and the actual 8 transmitted bytes are:

```text
00 10 11 02 22 01 00 00
```

\---

# Why Item IDs Matter

After the 16-packet warm-up, the test intentionally alternates item order.

For one index the PC may send:

```text
\[index]\[ITEM\_A]\[price\_A]\[ITEM\_B]\[price\_B]
```

and for another:

```text
\[index]\[ITEM\_B]\[price\_B]\[ITEM\_A]\[price\_A]
```

This checks whether your FPGA routes by **ID** rather than assuming:

```text
slot 1 = Item A
slot 2 = Item B
```

That assumption is incorrect.

If the PC sends:

```text
\[index]\[ITEM\_B]\[price\_B]\[ITEM\_A]\[price\_A]
```

the FPGA must return:

```text
\[index]\[ITEM\_B]\[action\_B]\[ITEM\_A]\[action\_A]\[reserved]
```

The response packet follows the **same item ordering as that input transaction**.

\---

# Software Reference Model

Before UART transmission begins, the script generates all 100 prices for Item A and all 100 prices for Item B.

It then runs those prices through two independent software reference models:

```text
Item A -> MovingAverageReference A
Item B -> MovingAverageReference B
```

The expected FPGA actions are therefore calculated **before the hardware test begins**.

The FPGA's answers are compared against this precomputed reference.

\---

# 16-Sample Moving Average

Each item maintains its own 16-sample window.

During the first 16 samples:

```text
index 0 through index 15
```

the software only fills the history.

These packets are treated as:

```text
IGNORED\_WARMUP
```

and are not included in correctness scoring.

Once the window is full, the reference computes:

```text
old\_average = running\_sum >> 4
```

Since the window contains 16 values:

```text
running\_sum / 16 = running\_sum >> 4
```

for the nonnegative integer values used by this test.

For the incoming price:

```text
new\_sum = old\_sum - oldest\_price + new\_price
new\_average = new\_sum >> 4
```

This means the reference uses **integer division by 16**, not floating-point averaging.

\---

# BUY / SELL Decision Logic

After warm-up, the software checks for crossings.

## BUY

A BUY occurs when:

```text
previous\_price <= old\_average
AND
current\_price > new\_average
```

Then:

```text
action = BUY = 0x02
```

## SELL

A SELL occurs when:

```text
previous\_price >= old\_average
AND
current\_price < new\_average
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

\---

# Two Independent Histories

Item A and Item B must have independent algorithm state.

Conceptually:

```text
ITEM A:
    16-price history
    running sum
    previous price
    current held action

ITEM B:
    16-price history
    running sum
    previous price
    current held action
```

Do not mix the histories.

The item ID determines which state receives a particular price.

\---

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

\---

# What Counts as a Correct Packet

After warm-up, the script checks five things:

```text
1. returned index == transmitted index
2. returned item1 == transmitted item1
3. returned action1 == software expected action1
4. returned item2 == transmitted item2
5. returned action2 == software expected action2
```

A packet is counted as correct only if **all five checks pass**.

Therefore, returning the correct BUY/SELL decisions with the wrong item IDs or index still produces an incorrect packet.

\---

# Action Correctness vs Packet Correctness

The test reports two correctness measurements.

## Action Correctness

Each returned action is checked independently.

There are two scored actions per post-warm-up packet:

```text
action1
action2
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
```

Packet correctness is therefore stricter.

\---

# Timeout / Partial Packet Handling

The PC requests exactly 8 bytes:

```python
rx = ser.read(8)
```

If fewer than 8 bytes arrive before the timeout, the test records a timeout/partial response and stops.

The test intentionally stops after an incomplete packet because continuing could cause subsequent UART bytes to become misaligned with packet boundaries.

Therefore:

> Your FPGA should always transmit exactly one complete 8-byte response for each complete 8-byte request.

\---

# Latency Measurement

For each transaction:

```text
t0 = immediately before PC transmission
t1 = immediately after the complete 8-byte FPGA response
```

Then:

```text
latency\_us = (t1 - t0) / 1000
```

The final summary reports the average latency for successfully received packets.

This is a **round-trip system measurement** and includes serial/USB/host overhead in addition to FPGA processing.

\---

# CSV Output

The test creates:

```text
trade\_results\_100.csv
```

The CSV records fields including:

```text
index

tx\_item1
tx\_price1
tx\_item2
tx\_price2

expected\_action1
expected\_action2

rx\_index
rx\_item1
rx\_action1
rx\_item2
rx\_action2
rx\_reserved

action1\_correct
action2\_correct
packet\_correct

status
latency\_us
```

This file is useful for debugging because it shows exactly which portion of a failed transaction did not match.

Possible status information can identify failures such as:

```text
INDEX
ITEM1
ACTION1
ITEM2
ACTION2
TIMEOUT
```

\---

# Summary TXT Output

The test also creates:

```text
trade\_summary\_100.txt
```

It reports:

* Requested packet count.
* Successfully received packet count.
* Number of ignored warm-up packets.
* Number of scored packets.
* Correct packet count.
* Packet correctness percentage.
* Correct individual action count.
* Action correctness percentage.
* Timeout count.
* Average successful round-trip latency.
* UART port.
* UART baud rate.
* Random seed.

This provides a compact final test result.

\---

# Expected Test Sequence

Conceptually, the test performs:

```text
1. Generate 100 random prices for Item A.
2. Generate 100 random prices for Item B.

3. Run all prices through the software reference model.
4. Save the expected actions.

5. Open UART at 115200 baud.

6. For index = 0 through 99:

      obtain Item A and Item B prices

      determine this transaction's item ordering

      construct exactly 8 input bytes

      start latency timer

      transmit input packet

      wait for exactly 8 output bytes

      stop latency timer

      if response is incomplete:
          record timeout
          stop test

      decode response

      if index < 16:
          mark as warm-up
      else:
          compare index
          compare item IDs
          compare actions
          score result

      save CSV row

7. Calculate final statistics.
8. Write CSV.
9. Write summary TXT.
10. Print final results.
```

\---

# What Participants May Change

Normally, participants only need to change the serial port:

```python
PORT = "COM6"
```

For example:

```python
PORT = "COM4"
```

depending on the port assigned by Windows.

\---

# What Participants Must NOT Change

Do **not** modify these competition protocol values:

```text
ITEM\_A = 0x11
ITEM\_B = 0x22

NONE = 0x00
SELL = 0x01
BUY  = 0x02
```

Do not create a separate HOLD encoding. Holding means retaining the previous action value.

Do **not** modify the input packet:

```text
\[index16]\[item1\_8]\[price1\_16]\[item2\_8]\[price2\_16]
```

Do **not** modify the output packet:

```text
\[index16]\[item1\_8]\[action1\_8]\[item2\_8]\[action2\_8]\[reserved16]
```

Do **not** reorder fields.

Do **not** force Item A and Item B into fixed packet slots.

Do **not** change the packet size from 8 bytes.

Do **not** return ASCII text.

Do **not** change the byte order.

Do **not** change the expected BUY/SELL algorithm in the test script in order to make an incompatible FPGA result appear correct.

\---

# Common FPGA Implementation Mistakes

## 1\. Reversing the action codes

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

\---

## 2\. Treating slot 1 as permanently Item A

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

\---

## 3\. Returning items in a fixed A/B order

Wrong if the request was B/A:

```text
response = A/actionA, B/actionB
```

Required:

```text
request  = B/priceB, A/priceA
response = B/actionB, A/actionA
```

\---

## 4\. Using floating-point averaging

The reference uses:

```text
average = running\_sum >> 4
```

Match this integer behavior.

\---

## 5\. Returning NONE whenever there is no crossing

The reference holds the previous action.

Wrong:

```text
no crossing -> NONE
```

Required:

```text
no crossing -> previous action remains unchanged
```

\---

## 6\. Responding before the entire request is decoded

Wait until the complete 8-byte input packet has been received and reconstructed before processing it.

\---

## 7\. Sending an incorrect number of response bytes

The PC expects:

```text
8 bytes exactly
```

An incomplete response can terminate the test.

\---

# Final Protocol Cheat Sheet

```text
UART
----
Baud = 115200

ITEM IDs
--------
ITEM\_A = 0x11 = 00010001
ITEM\_B = 0x22 = 00100010

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

\[index16]\[item1\_8]\[price1\_16]\[item2\_8]\[price2\_16]

Bytes:
0 index\[15:8]
1 index\[7:0]
2 item1
3 price1\[15:8]
4 price1\[7:0]
5 item2
6 price2\[15:8]
7 price2\[7:0]

FPGA -> PC
----------
64 bits / 8 bytes

\[index16]\[item1\_8]\[action1\_8]\[item2\_8]\[action2\_8]\[reserved16]

Bytes:
0 index\[15:8]
1 index\[7:0]
2 item1
3 action1
4 item2
5 action2
6 reserved\[15:8]
7 reserved\[7:0]

MULTI-BYTE ORDER
----------------
Big-endian

MOVING AVERAGE
--------------
Window = 16 samples
Indices 0-15 = warm-up / not scored
average = running\_sum >> 4

BUY
---
previous\_price <= old\_average
AND
current\_price > new\_average

SELL
----
previous\_price >= old\_average
AND
current\_price < new\_average

OTHERWISE
---------
retain previous action

TRANSPORT
---------
stop-and-wait:
send one 8-byte request
wait for one 8-byte response
then send next request
```

\---

# Competition Rule Summary

The testing scripts define the external hardware/software interface.

**Participants implement the algorithm and hardware architecture. Participants do not redefine the packet protocol.**

If your FPGA produces a different packet format, item encoding, action encoding, field ordering, or byte ordering, the official tester will interpret those bytes according to the specification above and the result will be incorrect.

Design your FPGA to match the tester, not the tester to match your FPGA.
