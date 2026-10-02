# GQH Hardware Track — Final Submission Checklist

Use this checklist before completing the official **Devpost** submission at <https://gqhacks.devpost.com>.

**Deadline:** Sunday, October 4, 2026, 11:00 am EDT. Submit first, then drop off the board.

## Repository Access

- [ ] Correct team GitHub repository URL is ready.
- [ ] Judges can access the repository (public: link works when logged out of GitHub).
- [ ] No passwords, API keys, tokens, private keys, or other secrets are committed.

## Required Project Files

- [ ] Final HDL source files are included.
- [ ] The **organizer-supplied** `19_tang_nano_20k.cst` file is included.
- [ ] The team did **not** create its own constraint file or recreate pin assignments in FloorPlanner.
- [ ] Gowin project/build files needed to reproduce the design are included.
- [ ] The final generated **`.fs` file** is included, built from the exact submitted source.
- [ ] Any host-side code is for local testing/demo only; the design does not depend on host software during judging.

## README

- [ ] README identifies the top-level entity/module.
- [ ] README identifies the Gowin EDA version used.
- [ ] README reports the total LUT count from the Gowin synthesis report.
- [ ] Build instructions are complete.
- [ ] FPGA programming instructions are complete.
- [ ] External libraries, IP cores, starter code, and other pre-existing resources are disclosed.
- [ ] Known limitations or incomplete features are documented.

## Fixed Ports

- [ ] Top-level port names match `19_tang_nano_20k.cst` exactly and were not renamed: `sys_clk`, `reset_btn`, `uart_rx_i`, `uart_tx_o`, `led0_n`, `led1_n`.
- [ ] Unused optional ports remain in the top-level port list (unused LEDs driven high).

## Protocol Verification

- [ ] UART is configured for **115200 baud, 8N1, LSB first**.
- [ ] FPGA receives exactly **8 bytes** per request.
- [ ] FPGA returns exactly **8 bytes** per response, only after all 8 request bytes arrive, and never sends unsolicited bytes.
- [ ] Multi-byte packet fields are **big-endian**.
- [ ] `ITEM_A = 0x11`.
- [ ] `ITEM_B = 0x22`.
- [ ] `NONE = 0x00`.
- [ ] `SELL = 0x01`.
- [ ] `BUY = 0x02`.
- [ ] `reserved = 0x0000` (a nonzero value makes the packet incorrect).
- [ ] Returned index matches received index.
- [ ] Items are routed by item ID only (never by slot or index), and the response mirrors the request's slot order.
- [ ] No separate HOLD packet code was introduced.

## Algorithm

- [ ] Index 0 clears all state, then its prices are processed as the first samples of the new window.
- [ ] Warm-up (indices 0–15) fills the window and sum, updates the previous price, and responds `NONE` for both actions.
- [ ] Averages use `sum >> 4` (floor, not rounded).
- [ ] When no crossing occurs, the item's last action is repeated.

## Testing

- [ ] `21_quick_uart_test.py` prints `PASS` on the board.
- [ ] `22_robust_uart_test.py` runs all 100 packets with no timeouts, and `trade_results_100.csv` has been reviewed.
- [ ] The design adds idle time or buffering between response bytes so the BL616 bridge does not drop or corrupt bytes.
- [ ] Final design has been tested on the Tang Nano 20K.
- [ ] Team understands the official run: **100 packets** (indices 0–99), warm-up indices 0–15, **84 scored packets / 168 scored actions**, 1-second per-packet timeout.

## Judging Metrics

- [ ] **Correctness (70):** packet correctness 50 + action correctness 20. A packet counts as correct only if the index, both item IDs, both actions, and `reserved = 0x0000` are all correct.
- [ ] **Latency (15):** average round-trip latency vs. the 16.626 ms reference (≤ 1.25× → 15, ≤ 2× → 8).
- [ ] **LUT usage (15):** total LUT count from **Synthesis Report → Resource → Resource Usage Summary**, vs. the 542-LUT reference.
- [ ] Team understands latency and LUT points are 0 if packet correctness is below 95%.

## Freeze the Final Version

- [ ] All final changes are committed.
- [ ] All final changes are pushed to GitHub.
- [ ] Full final commit SHA has been copied.

```bash
git rev-parse HEAD
```

- [ ] The final commit SHA corresponds to the exact version intended for judging.

## Devpost

- [ ] Project information is complete.
- [ ] GitHub repository URL is included.
- [ ] Full final commit SHA is included in your Devpost submission (in the project description if there is no dedicated field).
- [ ] The SHA is **not** in the README: a commit cannot contain its own SHA.
- [ ] Devpost submission is complete before **11:00 am EDT on October 4**.

## Drop-off

- [ ] Board and accessories dropped off at Reitz Room 2345 by 11:00 am on Oct 4.
