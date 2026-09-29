# GQH Hardware Track — Final Code Submission

This repository contains the **official code-submission requirements, recommended repository structure, and README template** for teams participating in the GQH Hardware Track FPGA Hackathon.

> **Important:** Each team should submit its **own GitHub repository**. This repository is the shared instructions/template repository and should not be used to upload all teams' code.

## Quick Start

Before the submission deadline, each team should:

1. Create a GitHub repository for the team project.
2. Make sure the repository contains the final HDL/source code and the files needed to build the FPGA project.
3. Copy the structure from [TEAM_README_TEMPLATE.md](TEAM_README_TEMPLATE.md) into the team's own `README.md` and complete every applicable section.
4. Review the recommended layout in [REPOSITORY_STRUCTURE.md](REPOSITORY_STRUCTURE.md).
5. Confirm that organizers/judges can access the repository.
6. Create and push a **final commit** representing the submitted version.
7. Copy the **full Git commit SHA** for that final commit.
8. Complete the official GQH Hardware Track submission form using the repository URL and final commit SHA.
9. Review [SUBMISSION_CHECKLIST.md](SUBMISSION_CHECKLIST.md) before submitting.

## What the Team Repository Should Contain

At minimum, the submitted repository should contain:

- Final HDL/source code used for the project
- Tang Nano 20K constraint file(s), including the relevant `.cst` file
- Project/build files needed to reproduce the design
- A complete `README.md` explaining how to build, program, and test the project
- The top-level entity/module name
- The toolchain and version used
- Any host-side software required to run the project, if applicable
- A disclosure of external libraries, IP cores, starter code, or other pre-existing resources used

Teams should also include testbenches, generated bitstreams, benchmark files, or other artifacts **when applicable or when required by the organizers**.

## Recommended Repository Layout

```text
team-project/
├── README.md
├── src/
│   └── HDL source files
├── constraints/
│   └── Tang Nano 20K .cst file(s)
├── testbench/
│   └── simulation/testbench files
├── gowin/
│   └── Gowin project/build files
├── host/
│   └── optional Python/C/C++/other host-side code
├── bitstream/
│   └── generated FPGA bitstream, if required
└── results/
    └── optional benchmarks, plots, logs, or output data
```

See [REPOSITORY_STRUCTURE.md](REPOSITORY_STRUCTURE.md) for details.

## Final Commit SHA

The submission form asks for the **full Git commit SHA** so organizers can identify the exact version submitted before the deadline.

Before submitting:

```bash
git add .
git commit -m "Final hackathon submission"
git push origin main
git rev-parse HEAD
```

The final command prints a value similar to:

```text
7fe929310cd84d0e1f1d6c1234567890abcdef12
```

Copy the **entire SHA** into the submission form.

If your default branch is not `main`, push the branch your team is using and make sure the submitted SHA points to the exact final version.

## Repository Access

Teams are responsible for making sure organizers and judges can access the submitted repository.

- **Public repository:** no additional access step is normally necessary.
- **Private repository:** follow the organizer-provided instructions for granting judging access before the deadline.

Do not put passwords, API keys, tokens, private keys, or other secrets in the repository.

## Reproducibility

A judge or organizer should be able to open your repository and understand:

- What the project does
- Which parts run directly on the FPGA
- Which HDL/source files are part of the final design
- The top-level entity/module
- Which version of the toolchain was used
- How to synthesize and complete place & route
- How to program the Tang Nano 20K
- How to reproduce the team's demo or expected output

Use [TEAM_README_TEMPLATE.md](TEAM_README_TEMPLATE.md) to make sure these items are documented.

## Submission Freeze

The repository URL may continue to exist after the deadline, but evaluation should be based on the **final commit SHA submitted through the official form**. Changes pushed after the deadline may not be considered during judging.

## Submission Form

The official Google Form link will be added here once finalized.

**Submission Form:** _To be added_

## Questions

For event-specific questions, contact the GQH Hardware Track organizers through the official hackathon communication channel.

---

### Organizer note

Some requirements may be updated before the event begins. Teams should use the latest version of these instructions and any official announcements from the GQH Hardware Track organizers.
