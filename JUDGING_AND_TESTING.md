# GQH Hardware Track — Judging & Testing Rules

This page summarizes how the official judging run works, how teams are scored, the fixed top-level ports, the fixed PC ↔ FPGA packet protocol, and the exact moving-average algorithm.

It follows the [GQH Hardware Track Participant Guide](participant-resources/GQH_Hardware_Track_Participant_Guide.pdf). If anything here disagrees with the guide, the guide wins.

> **No host-side computation.** During the official run, **no team-supplied host software is executed**. All UART parsing, state, algorithmic computation, and response generation must happen **on the FPGA**. Host-side code in a team repository is for local testing or demos only.

> **Warning: the BL616 USB-serial bridge.** The Tang Nano 20K's onboard BL616 USB-serial bridge can **drop or corrupt bytes** if the FPGA sends response bytes back-to-back with no idle time. The judge then logs a **TIMEOUT**. A design whose logic is functionally correct can still fail this way.
>
> Your design **must** add idle time or buffering between response bytes. The latency the judge measures **includes** any delay you add, so there is a trade-off: more idle time is safer but slower. Test with `22_robust_uart_test.py` and keep the CSV.

---

# Judging Flow

1. Teams submit through Devpost (repository URL + full commit SHA) by **Sunday, October 4, 2026, 11:00 am EDT**.
2. Teams then return the board and all accessories to **Reitz Room 2345** by **11:00 am on Sunday, October 4**.
3. Judges program each board in **SRAM mode** with the `.fs` file from the team's **submitted commit**. The `.fs` file must be in the team repository.
4. The official test runs on a **single judging PC**, with Gowin Programmer and any serial terminals closed before the judge opens the COM port.
5. Each team is judged on **one official run**.
6. If a run fails for a reason outside the design (for example a cable or the wrong COM port), judges may **reprogram and rerun once**.

Teams do not need to keep the board powered or connected. The submission must be final before drop-off; changes pushed afterward are not judged.

---

# The Official Run

| Item | Value |
|---|---|
| Packets | **100**, indices `0`–`99` |
| Warm-up | Indices `0`–`15` (not scored for correctness) |
| Scored | Indices `16`–`99`: **84 packets**, **168 actions** |
| Transport | Stop-and-wait |
| Per-packet timeout | **1 second** |
| Price seed | Chosen by the organizers, the **same for every team**, and **not published in advance** |

- **Stop-and-wait:** the judge sends one 8-byte request and waits for the complete 8-byte response before sending the next.
- **Index 0 starts a new session.** The board is not reset or reprogrammed between runs, so your design must clear all previous state by itself when it receives index 0 (see [the algorithm](#exact-16-sample-moving-average-algorithm)).
- **Warm-up packets** are not scored for correctness, but their prices still count toward the average, and their responses must still follow the protocol and arrive in time.
- **Timeout:** a packet whose 8 response bytes do not all arrive within 1 second is logged as `TIMEOUT`. It counts as incorrect and **ends the run**; packets that are never received score zero.
- **Price seed:** because the seed is not published, do not hardcode price patterns.

---

# Scoring (100 points)

Correctness is worth 70 points (packet 50 + action 20), latency 15, and LUT usage 15.

| Criterion | Points | Measured from | How points are awarded |
|---|---:|---|---|
| Packet correctness | 50 | Packets with the right index, both item IDs, and both actions, out of **84** scored packets | `50 × correct packets ÷ 84` |
| Action correctness | 20 | Individual actions, out of **168** scored actions | `20 × correct actions ÷ 168` |
| Latency | 15 | Average round-trip latency of received packets, compared with the organizer reference design on the same judge PC | 15 if at most 1.25× the reference average; 8 if at most 2×; otherwise 0 |
| LUT usage | 15 | Total LUT count in the Gowin synthesis report, compared with the reference design | `15 × min(1, reference LUTs ÷ your LUTs)` |
| **Total** | **100** | | |

> **Correctness gate:** latency and LUT points are **0** if packet correctness is below **95%** (fewer than 80 of the 84 scored packets correct).

The denominators are fixed. A packet that times out, or is never sent because the run ended, counts as incorrect.

## Reference Values

| Reference | Value |
|---|---|
| Reference average latency | **16.626 ms** |
| 1.25× reference (15 latency points) | about **20.8 ms** |
| 2× reference (8 latency points) | about **33.3 ms** |
| Reference LUTs | **542** — the average of the VHDL and Verilog reference designs, from the `LUT` line of **Gowin Synthesis Report → Resource → Resource Usage Summary** |

## Latency Definition

Latency is measured from **just before the judge transmits a request** until **the complete 8-byte response has been received**:

```text
latency = time_after_complete_response - time_before_transmit
```

It includes:

- UART transfer PC → FPGA (8 bytes)
- Your FPGA's processing
- Any idle time your design adds between response bytes
- UART transfer FPGA → PC (8 bytes)
- USB/serial overhead on the PC

At 115200 baud (8N1), the 16 bytes of one transaction need only about **1.39 ms** of wire time. Most of the roughly 16.6 ms measured on the organizers' test PC is **BL616, USB, operating-system, and serial-buffering overhead**, not the UART itself or your logic. Expect only small differences between correct designs.

## LUT Usage

After synthesis in Gowin EDA:

1. In the **Process** pane, double-click **Synthesis Report**.
2. Open **Resource → Resource Usage Summary**.
3. Record the total **LUT** usage.
4. Gowin also shows a breakdown such as `LUT2`, `LUT3`, `LUT4`, etc. The **total** is the number used for judging.

Report your total LUT count in your team README.

---

# Fixed Top-Level Ports

Top-level port names must match the organizer-supplied `19_tang_nano_20k.cst` **exactly**. The names are fixed for this competition and **may not be renamed**.

| Port | FPGA pin | Direction | Purpose |
|---|---:|---|---|
| `sys_clk` | 4 | in | On-board 27 MHz clock |
| `reset_btn` | 87 | in | User reset button, pull-down (optional) |
| `uart_rx_i` | 70 | in | BL616 → FPGA (UART input) |
| `uart_tx_o` | 69 | out | FPGA → BL616 (UART output) |
| `led0_n` | 15 | out | Active-low status LED (optional) |
| `led1_n` | 16 | out | Active-low status LED (optional) |

- If you do not use an optional port, keep it in your top-level port list and leave it unconnected (drive an unused LED high).
- The **top-level entity/module name is your choice**. Set it in Gowin under **Project → Configuration → Synthesize → General → Top Module/Entity**.
- Add the organizer-supplied `.cst` to your Gowin project as the physical constraint file. Do not create your own `.cst` and do not recreate the pin assignments in FloorPlanner.

---

# Fixed UART Packet Protocol

> **The packet protocol is not participant-configurable. Do not change it to match your implementation. Your FPGA implementation must match the official tester.**

UART:

```text
Baud rate      = 115200
Frame          = 8 data bits, no parity, 1 stop bit (8N1)
Bit order      = LSB first
Packet size    = 8 bytes in each direction
Multi-byte     = big-endian (most-significant byte first)
Prices         = unsigned 16-bit
```

## PC → FPGA Request

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

## FPGA → PC Response

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

The index and item fields echo the request. `reserved` **must** be:

```text
reserved = 0x0000
```

## Response Rules

- Send **exactly one** 8-byte response for every 8-byte request.
- **Never** send unsolicited bytes.
- **Never** send a response before all 8 request bytes have arrived.
- The response mirrors the request's slot order (see below).

## Fixed Item IDs

```text
ITEM_A = 0x11 = 00010001
ITEM_B = 0x22 = 00100010
```

The item IDs are fixed for the entire competition. The item ID determines which independent item state receives the associated price.

## Fixed Action Codes

```text
NONE = 0x00 = 00000000
SELL = 0x01 = 00000001
BUY  = 0x02 = 00000010
```

There is **no separate HOLD code**. When no crossing occurs, send that item's last action again. `NONE` is sent only before that item's first crossing.

## Item Routing and Slot Order

The tester may place **either item in either slot on any packet**. Route exclusively by **item ID**, never by packet index or slot position.

The response mirrors the request's slot order. If the request is:

```text
[index][ITEM_B][price_B][ITEM_A][price_A]
```

the response must be:

```text
[index][ITEM_B][action_B][ITEM_A][action_A][reserved]
```

Do not assume slot 1 is always Item A.

## Worked Example

Request (judge → FPGA): index 16, Item A price 80, Item B price 200.

```text
00 10 11 00 50 22 00 C8
```

| Bytes | Field | Hex | Meaning |
|---|---|---|---|
| 0–1 | index | `00 10` | 16 |
| 2 | item1 | `11` | Item A |
| 3–4 | price1 | `00 50` | 80 |
| 5 | item2 | `22` | Item B |
| 6–7 | price2 | `00 C8` | 200 |

A possible response (FPGA → judge): Item A = SELL, Item B = BUY.

```text
00 10 11 01 22 02 00 00
```

| Bytes | Field | Hex | Meaning |
|---|---|---|---|
| 0–1 | index | `00 10` | 16, echoed |
| 2 | item1 | `11` | Item A, echoed |
| 3 | action1 | `01` | SELL |
| 4 | item2 | `22` | Item B, echoed |
| 5 | action2 | `02` | BUY |
| 6–7 | reserved | `00 00` | must be zero |

## Stop-and-Wait Transport

The judge sends one transaction and waits for its complete response before sending the next:

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

---

# Exact 16-Sample Moving-Average Algorithm

## Per-Item State

Each item maintains **completely independent** state:

- A window of its last **16** prices
- The **sum** of those 16 prices (16 unsigned 16-bit prices need **20 bits**)
- The **previous price**
- The **last action** (`NONE` until its first crossing)

## Reset (Index 0)

Index 0 starts a new session. On receipt of index 0, **first** clear all previous session state for both items (every window, sum, previous price, and last action), **then** process index 0's two prices as the first samples of the new window. Your design must start a fresh session on index 0 by itself.

## Warm-Up (Indices 0–15)

Indices 0–15 only fill the first window. For each warm-up packet, for each item:

1. Add the item's price to its window and sum.
2. Store that price as the item's previous price, so that at index 16 the previous price is the price from index 15.

No crossing is evaluated during warm-up. Respond to warm-up packets with `NONE` for both actions.

## Update Procedure (Index 16 Onward)

Perform these steps independently for each item:

1. Old average: `old_average = old_sum >> 4` — floor division by 16. The fraction is discarded, not rounded.
2. Slide the window: `new_sum = old_sum - oldest_price + current_price`.
3. New average: `new_average = new_sum >> 4`.
4. **BUY** if `previous_price <= old_average` **AND** `current_price > new_average`.
5. **SELL** if `previous_price >= old_average` **AND** `current_price < new_average`.
6. Otherwise no crossing has happened: repeat the item's last action.

Then store the current price as the new previous price.

The previous price is compared against the **old** average, and the current price against the **new** average. The current price is already included in the new average when the second comparison is made.

| Previous price | Current price | Action |
|---|---|---|
| at or below the old average | above the new average | **BUY** — upward crossing |
| at or above the old average | below the new average | **SELL** — downward crossing |
| any other case | | Repeat the last action; no crossing |

Example — Item B, the average is 100:

| Before | Now | Action | Comment |
|---:|---:|---|---|
| 95 | 105 | BUY | Crossed up |
| 105 | 94 | SELL | Crossed down |
| 105 | 120 | Repeat last action | No crossing |

## Same Steps as Pseudocode

```text
on index 0:
    for both items: clear window, sum, previous_price; last_action = NONE

for each (item_id, price) in the request, routed by item_id:
    s = state[item_id]
    if index <= 15:                                  # warm-up
        add price to s.window; s.sum = s.sum + price
        s.previous_price = price
        action = NONE
    else:
        old_average = s.sum >> 4
        new_sum     = s.sum - s.oldest_price + price
        new_average = new_sum >> 4
        if s.previous_price <= old_average and price > new_average:
            s.last_action = BUY
        elif s.previous_price >= old_average and price < new_average:
            s.last_action = SELL
        # otherwise s.last_action is unchanged
        replace s.oldest_price with price in s.window; s.sum = new_sum
        s.previous_price = price
        action = s.last_action
    put (item_id, action) in the same slot it arrived in
```

---

# Testing Resources

Use the tests in this order:

1. **Quick UART test** (`21_quick_uart_test.py`) — basic communication and packet-format verification.
2. **Robust UART test** (`22_robust_uart_test.py`) — scoring-style test with a software reference model.

Both scripts need **Python 3** and **pyserial** (`pip install pyserial`).

The robust test runs **100 packets** (indices 0–99) with the same warm-up and scoring structure as the official run: 84 scored packets and 168 scored actions, a 1-second per-packet timeout, and stop-and-wait transport. It uses a **placeholder participant seed, not the official judging seed**, and after warm-up it randomly places each item in either slot (seeded, so runs are reproducible). It saves a CSV of every packet. Keep it: it is the best debugging tool you have.

Reference documentation:

- [21_quick_uart_test_REFERENCE.md](participant-resources/testing/21_quick_uart_test_REFERENCE.md)
- [22_robust_uart_test_REFERENCE.md](participant-resources/testing/22_robust_uart_test_REFERENCE.md)

> **[TODO]** `21_quick_uart_test.py` and `22_robust_uart_test.py` are not yet in this repository. Add them to `participant-resources/testing/`.

## Important

Change only the `PORT` setting in the scripts to your board's COM port.

Do **not** change the packet protocol or the scoring logic to make a design pass.
