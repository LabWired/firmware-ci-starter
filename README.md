# firmware-ci-starter

A starting point for testing firmware in CI with LabWired, without hardware. Use this
template and the first push already runs a firmware simulation in GitHub Actions.

## What's in here

`firmware/` is a small ARM Cortex-M0 program (`firmware/src/main.c`) that writes a
greeting over UART and then loops. `firmware/target/firmware` is that program already
built, so CI does not need a cross-compiler to run the test. `tests/firmware-regression.yaml`
runs the firmware in LabWired's simulator for 20000 steps and checks that the UART
output contains the greeting. `.github/workflows/labwired.yml` runs that test on every
push and pull request using the `labwired/firmware-test` action.

## Using this for your own firmware

Replace `firmware/target/firmware` with your own ELF, or point
`tests/firmware-regression.yaml` at wherever your build puts it. Update the `system`
field in the same file if your board isn't the generic Cortex-M0 used here. Change
the `uart_contains` string to something your firmware actually prints, and add more
assertions as needed — see the LabWired test script reference for what's available
(register checks, memory checks, stop conditions, coverage).

If you rebuild the firmware in this repo, add a step to `.github/workflows/labwired.yml`
that runs `make -C firmware` before the LabWired step, so CI always tests what your
source currently produces rather than a stale binary. The tradeoff of committing a
prebuilt ELF is exactly that: it can drift from the source if you forget to rebuild
and recommit. For a real project, building in CI is usually worth the extra runner
minutes.

## What you get back

Pull requests get a comment with pass/fail status, step and cycle counts, and the
UART output. Every run also gets a hosted report on labwired.com, and (for public
repositories) shows up in the public gallery.

## Rebuilding the firmware locally

```
cd firmware
make
```

Requires `arm-none-eabi-gcc`. `main.c` is a bare Cortex-M0 program: a two-entry vector
table, a reset handler that calls `main`, and a UART driver that writes one register
per character. LabWired's simulation is instruction-level and silicon-validated
against real STM32 hardware, not an abstract model, so what passes here reflects how
the instructions actually execute.
