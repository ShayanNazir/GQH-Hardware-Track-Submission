# Recommended Team Repository Structure

This document explains the recommended layout for GQH Hardware Track team repositories.

The exact internal organization may vary, but judges should be able to quickly locate the final source code, organizer-supplied constraints, build files, and reproduction instructions.

## Suggested Layout

```text
team-project/
├── README.md
├── src/
│   └── HDL/VHDL source files
├── constraints/
│   └── organizer-supplied Tang Nano 20K .cst
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

Use [TEAM_README_TEMPLATE.md](TEAM_README_TEMPLATE.md) as a starting point.

The README should explain:

- What the project does
- What runs directly on the FPGA
- Hardware/toolchain used
- Top-level entity/module
- How to build the design
- How to program the Tang Nano 20K
- How to reproduce the demo
- Expected inputs and outputs
- Testing results
- LUT usage
- External resources used
- Known limitations

## `src/`

Store the HDL/VHDL source files that make up the FPGA design.

## `constraints/`

Use the **organizer-supplied Tang Nano 20K `.cst` file**.

Participants do **not** need to create a new constraint file in Gowin FloorPlanner. Add the supplied `.cst` to the project as the physical constraint file and make sure the top-level port names match it.

## `testbench/`

Store simulation/testbench files here when applicable.

If the project does not use a testbench, explain the hardware verification procedure in the README.

## `gowin/`

Store project/build files required to reproduce the design in Gowin EDA.

Avoid committing unnecessary generated caches or machine-specific temporary files.

## `host/`

Use this directory for any team-created host software that communicates with the FPGA.

The official organizer testing scripts do not need to be copied into every team repository unless organizers specifically request it; they are distributed from the central GQH resource repository.

## `bitstream/`

If organizers require the generated FPGA programming file, store the final file here.

## `results/`

Optional location for:

- Correctness/test outputs
- Latency measurements
- Gowin resource-utilization information
- LUT usage
- Logs
- Plots
- Demo output

## Do Not Commit Secrets

Never commit passwords, API keys, access tokens, private SSH keys, or personal credentials.

## Final Submission Version

The final judged version should correspond to the **full commit SHA identified in the team's Devpost submission or other organizer-specified submission field**.

Before submitting:

```bash
git status
git add .
git commit -m "Final hackathon submission"
git push
git rev-parse HEAD
```

Verify that the final commit is pushed successfully.
