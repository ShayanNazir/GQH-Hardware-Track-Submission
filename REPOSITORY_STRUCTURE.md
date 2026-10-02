# Recommended Team Repository Structure

This document explains the recommended layout for GQH Hardware Track team repositories. It follows Part 3 of the [participant guide](participant-resources/GQH_Hardware_Track_Participant_Guide.pdf).

Each team submits its **GitHub repository URL and full commit SHA** through Devpost. There is no zip upload. The exact internal organization may vary, but judges should be able to quickly locate the final source code, organizer-supplied constraints, build files, the final `.fs` file, and reproduction instructions.

The repository **must be public**; private repositories are not accepted. Keep it public through judging, do not delete or rename it, and verify that its link opens while you are logged out of GitHub.

## Suggested Layout

```text
team-project/
├── README.md
├── src/
│   └── HDL source files
├── constraints/
│   └── 19_tang_nano_20k.cst (organizer-supplied)
├── testbench/
│   └── testbench/simulation files
├── gowin/
│   └── Gowin project/build files
├── bitstream/
│   └── final .fs file (required)
├── host/
│   └── optional host-side test/demo code (not run during judging)
└── results/
    └── optional test CSVs and benchmarks
```

## `README.md`

Use [TEAM_README_TEMPLATE.md](TEAM_README_TEMPLATE.md) as a starting point.

The README should let a judge understand and reproduce the project without guessing. It must include:

- Team/project name and team members
- Brief project description
- What functionality is implemented directly on the FPGA
- Any host-side tooling used for local testing (host software is not run during judging)
- FPGA board: Tang Nano 20K
- HDL/languages used
- Gowin EDA version used
- Top-level entity/module name
- Build instructions and FPGA programming instructions
- Project inputs and expected outputs; how to reproduce the final demo
- Testing/verification procedure
- Relevant performance results, including your total LUT count
- External libraries, IP cores, starter code, datasets, or other pre-existing resources used
- Known limitations or incomplete features

## `src/`

Store the HDL source files that make up the FPGA design.

## `constraints/`

Use the **organizer-supplied `19_tang_nano_20k.cst` file**.

Participants do **not** create their own constraint file or recreate pin assignments in Gowin FloorPlanner. Add the supplied `.cst` to the project as the physical constraint file. Top-level port names must match it exactly and may not be renamed.

## `testbench/`

Store simulation/testbench files here when applicable. Do not add simulation-only testbenches to the synthesis sources.

If the project does not use a testbench, explain the hardware verification procedure in the README.

## `gowin/`

Store project/build files required to reproduce the design in Gowin EDA.

Avoid committing unnecessary generated caches or machine-specific temporary files. The final `.fs` file is the exception: it is required (see below).

## `bitstream/`

**Required.** Store the final generated `.fs` file here, built from the exact source in the submitted commit. Gowin writes it to `impl/pnr/<project>.fs`. Judges program the board in SRAM mode from this file.

## `host/`

Optional. Use this directory for any team-created host software used for **local testing or demos only**.

During the official run, no team-supplied host software is executed. All UART parsing, state, computation, and response generation must happen on the FPGA.

The official organizer testing scripts do not need to be copied into every team repository unless organizers specifically request it; they are distributed from this GQH resource repository.

## `results/`

Optional location for:

- `trade_results_100.csv` and `trade_summary_100.txt` from `22_robust_uart_test.py`
- Latency measurements
- Gowin resource-utilization information, including total LUT usage
- Logs
- Plots
- Demo output

## Do Not Commit Secrets

Never commit passwords, API keys, access tokens, private SSH keys, or personal credentials.

## Final Submission Version

The judged version is the **full commit SHA identified in the team's Devpost submission**.

Before submitting:

```bash
git status
git add .
git commit -m "Final hackathon submission"
git push
git rev-parse HEAD
```

Verify that the final commit is pushed successfully, then enter the repository URL and the full SHA from `git rev-parse HEAD` on Devpost (in your Devpost project description if there is no dedicated field).

Do not put the SHA in your README: a commit cannot contain its own SHA.
