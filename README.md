# Code Collection

Personal code collection for circuit design, verification, automation, data
processing, and engineering utilities.

## Directory structure

| Directory | Contents |
|---|---|
| `Verilog/` | RTL, testbenches, and digital verification code |
| `VerilogA/` | Verilog-A and Verilog-AMS behavioral models |
| `SKILL/` | Cadence Virtuoso SKILL utilities |
| `C/` | C programs and reusable modules |
| `Python/` | Python scripts and engineering tools |
| `MATLAB/` | MATLAB scripts, models, and data-analysis code |

## Available projects

### Verilog-A PRBS checker

Location: `VerilogA/prbs-checker/`

- External-clock PRBS checker.
- Data-driven automatic sampling-phase alignment checker.
- Parameterized PRBS7/9/13/15/23/31 support.
- Cadence `vsource` PN13 support using taps `13,4,3,1,0`.
- Lock detection, automatic re-acquisition, bit counting, and error counting.

See `VerilogA/prbs-checker/README.md` for parameters and usage.

## Notes

- Language categories are directories, not Git branches. Branches are reserved
  for feature development, experiments, and releases.
- Do not commit PDK files, proprietary models, license information, passwords,
  tokens, or confidential project data to this public repository.

