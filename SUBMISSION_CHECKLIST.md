# GQH Hardware Track — Final Submission Checklist

Use this checklist before completing the official **Devpost** submission.

## Repository Access

- [ ] Correct team GitHub repository URL is ready.
- [ ] Judges can access the repository.
- [ ] No passwords, API keys, tokens, private keys, or other secrets are committed.

## Required Project Files

- [ ] Final HDL/VHDL source files are included.
- [ ] The **organizer-supplied** Tang Nano 20K `.cst` file is included.
- [ ] The team did **not** unnecessarily recreate the board constraint file in FloorPlanner.
- [ ] Gowin project/build files needed to reproduce the design are included.
- [ ] README identifies the top-level entity/module.
- [ ] README identifies the toolchain/version used.
- [ ] Build instructions are complete.
- [ ] FPGA programming instructions are complete.
- [ ] External libraries, IP cores, starter code, and other pre-existing resources are disclosed.

## Protocol Verification

- [ ] UART is configured for **115200 baud**.
- [ ] FPGA receives exactly **8 bytes** per request.
- [ ] FPGA returns exactly **8 bytes** per response.
- [ ] Multi-byte packet fields are **big-endian**.
- [ ] `ITEM_A = 0x11`.
- [ ] `ITEM_B = 0x22`.
- [ ] `NONE = 0x00`.
- [ ] `SELL = 0x01`.
- [ ] `BUY = 0x02`.
- [ ] Returned index matches received index.
- [ ] Returned item order matches received item order.
- [ ] No separate HOLD packet code was introduced.

## Testing

- [ ] Quick UART test/reference has been reviewed and basic communication works.
- [ ] Robust UART test/reference has been reviewed.
- [ ] Final design has been tested on the Tang Nano 20K.
- [ ] Team understands that the official judging run uses **1000 input packets**.
- [ ] Team understands the first 16 samples are used for moving-average warm-up according to the reference test.

## Judging Metrics

- [ ] **Correctness:** outputs match the official reference.
- [ ] **Latency:** input-to-complete-output response delay is measured by the official UART tester.
- [ ] **LUT usage:** team knows where to find the total LUT count in Gowin:
      **Synthesis Report → Resource → Resource Usage Summary**.

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
- [ ] Final commit SHA is included if requested by the Devpost fields/organizers.
- [ ] Demo/video is included if required.
- [ ] Submission is completed before the official deadline.
