# GQH Hardware Track — Final Submission Checklist

Use this checklist before submitting the official Google Form.

## Team and Project Information

- [ ] Team/project name is finalized.
- [ ] Team member information is correct.
- [ ] FPGA number matches the board assigned during checkout.

## Repository Access

- [ ] The correct GitHub repository URL is ready.
- [ ] Organizers/judges can access the repository.
- [ ] If the repository is private, the required judging access has been granted.
- [ ] No passwords, API keys, tokens, private keys, or other secrets are committed.

## Required Project Files

- [ ] Final HDL/source files are included.
- [ ] Tang Nano 20K constraint `.cst` file(s) are included.
- [ ] Project/build files needed to reproduce the FPGA design are included.
- [ ] The repository README identifies the top-level entity/module.
- [ ] The README identifies the toolchain and version used.
- [ ] Build instructions are complete.
- [ ] FPGA programming instructions are complete.
- [ ] Host-side code is included if the project requires it.
- [ ] External libraries, IP cores, starter code, and other pre-existing resources are disclosed.

## When Applicable

- [ ] Testbench/simulation files are included.
- [ ] Generated bitstream is included if required by organizers.
- [ ] Benchmark or result files are included if relevant.
- [ ] Demo/reproduction instructions are included.
- [ ] Performance measurements explain how they were obtained.

## Final Verification

- [ ] The final project successfully synthesizes.
- [ ] The final project successfully completes place & route.
- [ ] The final project has been tested on the Tang Nano 20K.
- [ ] The repository README accurately describes the final project.
- [ ] The submitted demo/video corresponds to the final version, if applicable.

## Freeze the Final Version

- [ ] All final changes are committed.
- [ ] All final changes are pushed to GitHub.
- [ ] The full final commit SHA has been copied.

You can obtain the full SHA with:

```bash
git rev-parse HEAD
```

- [ ] The commit SHA entered in the submission form matches the exact final version to be judged.

## Official Form

- [ ] GitHub repository URL entered.
- [ ] Full final commit SHA entered.
- [ ] FPGA number entered correctly.
- [ ] All required form questions completed.
- [ ] Devpost/demo links entered if required.
- [ ] Final submission acknowledgments reviewed and accepted.
