# Recommended Team Repository Structure

This document explains the recommended layout for GQH Hardware Track team repositories.

The exact internal organization may vary by project, but judges should be able to quickly locate the final source code, constraints, build files, and instructions.

## Suggested Layout

```text
team-project/
├── README.md
├── src/
│   └── HDL source files
├── constraints/
│   └── Tang Nano 20K .cst file(s)
├── testbench/
│   └── testbench/simulation files
├── gowin/
│   └── Gowin project/build files
├── host/
│   └── optional host-side software
├── bitstream/
│   └── generated programming file, if required
└── results/
    └── optional benchmarks, logs, plots, or outputs
```

## `README.md`

This should be the first place a judge looks.

Use [TEAM_README_TEMPLATE.md](TEAM_README_TEMPLATE.md) as a starting point.

The README should explain:

- What the project does
- What runs directly on the FPGA
- Hardware and toolchain used
- Top-level entity/module
- How to build the design
- How to program the Tang Nano 20K
- How to reproduce the demo
- Expected inputs and outputs
- External resources used
- Known limitations

## `src/`

Store the HDL/source files that make up the FPGA design.

Examples:

```text
src/
├── top.vhd
├── order_book.vhd
├── signal_engine.vhd
└── uart_controller.vhd
```

or the equivalent files for another allowed HDL.

Avoid leaving abandoned experiments in the final source directory when possible.

## `constraints/`

Store the Tang Nano 20K pin/constraint files here.

Example:

```text
constraints/
└── tang_nano_20k.cst
```

The final constraint file should match the submitted design.

## `testbench/`

Store simulation and testbench files here when applicable.

Example:

```text
testbench/
├── top_tb.vhd
└── test_vectors/
```

If your project does not use a testbench, explain your hardware verification procedure in the README.

## `gowin/`

Store project/build files required to reproduce the design in Gowin EDA.

Keep generated caches or machine-specific temporary files out of Git when they are not required to reproduce the project.

## `host/`

Use this directory for software that runs on a host computer and communicates with the FPGA.

Examples:

```text
host/
├── send_market_data.py
├── collect_results.py
└── requirements.txt
```

If the project uses a host program, the README should clearly distinguish **host-side functionality** from **FPGA functionality**.

## `bitstream/`

If organizers require the generated FPGA programming file, store the final file here.

Do not rely on a bitstream as a replacement for source code and reproducible build instructions.

## `results/`

Optional location for:

- Benchmark output
- Latency/throughput measurements
- FPGA utilization reports
- Strategy/quant results
- Logs
- Plots
- Demo output

If results are used as part of judging, explain how they were generated.

## File Naming

Use descriptive names. Prefer:

```text
order_book.vhd
market_data_parser.vhd
strategy_engine.vhd
tang_nano_20k.cst
```

instead of:

```text
new.vhd
final2.vhd
thing.vhd
test123.vhd
```

## Do Not Commit Secrets

Never commit:

- Passwords
- API keys
- Access tokens
- Private SSH keys
- Personal credentials

If host-side software needs configuration, provide an example file such as:

```text
.env.example
```

with placeholder values instead of real credentials.

## Final Submission Version

The version evaluated should correspond to the **full commit SHA submitted through the official GQH Hardware Track form**.

Before submitting:

```bash
git status
git add .
git commit -m "Final hackathon submission"
git push
git rev-parse HEAD
```

Verify that `git status` is clean and that the final commit has been pushed successfully.
