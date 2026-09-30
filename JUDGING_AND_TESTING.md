# GQH Hardware Track — Judging & Testing Rules

This page summarizes the official technical judging criteria and the fixed PC ↔ FPGA interface.

## Judging Criteria

Teams will be evaluated on three technical criteria:

### 1. Output Correctness

The official judging run will use **1000 input packets**, not 100.

Correctness is determined using the official software reference and packet checks. The participant robust-test reference currently uses a smaller deterministic test for local verification, while the real judging run uses 1000 packets with a random seed.

The supplied reference test treats the first **16 samples as moving-average warm-up** and applies its scoring logic after that warm-up period.

A response must preserve the required transaction index and item ordering and return the expected actions.

### 2. Input-to-Output Delay / Latency

The official UART tester measures response delay from immediately before transmitting an input packet until the complete 8-byte FPGA response has been received.

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
- USB/serial/host overhead

### 3. LUT Usage

After synthesis in Gowin EDA:

1. In the **Process** pane, double-click **Synthesis Report**.
2. Open **Resource → Resource Usage Summary**.
3. Record the total **LUT** usage.
4. Gowin will normally also show a breakdown such as `LUT2`, `LUT3`, `LUT4`, etc.

The LUT number reported here is the resource-usage value used for judging.

---

# Fixed UART Packet Protocol

> **The packet protocol is not participant-configurable. Do not change it to match your implementation. Your FPGA implementation must match the official tester.**

UART:

```text
Baud rate = 115200
Packet size = 8 bytes in each direction
Multi-byte byte order = big-endian
```

## PC → FPGA Input

```text
[index16][item1_8][price1_16][item2_8][price2_16]
```

Total:

```text
16 + 8 + 16 + 8 + 16 = 64 bits = 8 bytes
```

Byte order:

```text
0  index[15:8]
1  index[7:0]
2  item1[7:0]
3  price1[15:8]
4  price1[7:0]
5  item2[7:0]
6  price2[15:8]
7  price2[7:0]
```

## FPGA → PC Output

```text
[index16][item1_8][action1_8][item2_8][action2_8][reserved16]
```

Total:

```text
16 + 8 + 8 + 8 + 8 + 16 = 64 bits = 8 bytes
```

Byte order:

```text
0  index[15:8]
1  index[7:0]
2  item1[7:0]
3  action1[7:0]
4  item2[7:0]
5  action2[7:0]
6  reserved[15:8]
7  reserved[7:0]
```

Recommended:

```text
reserved = 0x0000
```

## Fixed Item IDs

```text
ITEM_A = 0x11 = 00010001
ITEM_B = 0x22 = 00100010
```

The item ID determines which independent item history/state receives the associated price.

## Fixed Action IDs

```text
NONE = 0x00 = 00000000
SELL = 0x01 = 00000001
BUY  = 0x02 = 00000010
```

There is **no separate HOLD packet code**. If no new crossing occurs, the previously held BUY/SELL action remains unchanged.

## Preserve Received Item Order

The tester may swap Item A and Item B between packet positions.

If the request is:

```text
[index][ITEM_B][price_B][ITEM_A][price_A]
```

the response must preserve that order:

```text
[index][ITEM_B][action_B][ITEM_A][action_A][reserved]
```

Do not assume packet slot 1 is always Item A.

---

# Stop-and-Wait Transport

The PC sends one transaction and waits for its complete response before sending the next:

```text
PC sends packet N
        ↓
FPGA receives all 8 bytes
        ↓
FPGA processes packet N
        ↓
FPGA returns all 8 bytes
        ↓
PC receives response N
        ↓
PC sends packet N+1
```

The FPGA must send exactly one complete 8-byte response for every complete 8-byte request.

---

# Organizer-Supplied Constraint File

Participants **should not create a new board pin constraint file in FloorPlanner**.

The organizers will provide the required Tang Nano 20K `.cst` file. Add that file to the Gowin project as the physical constraint file.

Your top-level port names must match the names used by the supplied `.cst` file.

---

# Testing Resources

Use the tests in this order:

1. **Quick UART test** — basic communication and packet-format verification.
2. **Robust UART test** — functional verification and scoring-oriented testing.

Reference documentation:

- [21_quick_uart_test_REFERENCE.md](participant-resources/testing/21_quick_uart_test_REFERENCE.md)
- [22_robust_uart_test_REFERENCE.md](participant-resources/testing/22_robust_uart_test_REFERENCE.md)

The actual Python scripts will be added alongside these references when the organizers provide the final participant versions.

## Important

Participants may change the local serial/COM port as required by their computer.

Participants must **not** change the competition packet protocol or scoring logic in order to make an incompatible FPGA implementation pass.
