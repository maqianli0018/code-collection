# Verilog-A PRBS checker

`prbs_checker.va` is a PRBS checker intended for Spectre/AMS transient
simulations. It supports automatic initial phase acquisition or a fixed,
known seed.

## Pins

- `din`: analog data input.
- `clk`: sampling clock; a rising crossing of `VTH_CLK` samples `din`.
- `rst`: synchronous active-high reset, sampled at the clock edge.
- `err`: high for one sampled bit when that bit is wrong.
- `locked`: goes high when acquisition and `SKIP_BITS` are complete and
  checking has started.
- `bit_count`: checked-bit count encoded as voltage.
- `error_count`: error count encoded as voltage.

The count outputs use `COUNT_SCALE` volts/count. The default is `1 V/count`;
use a smaller value such as `1u` for long simulations.

## Polynomial and bit convention

The module sends/checks the current MSB, then advances the LFSR:

```text
expected_bit = lfsr[ORDER-1]
feedback     = lfsr[ORDER-1] XOR lfsr[TAP-1]
               XOR lfsr[TAP2-1] when TAP2 > 0
               XOR lfsr[TAP3-1] when TAP3 > 0
lfsr         = {lfsr[ORDER-2:0], feedback}
```

Use these common settings:

| Pattern | ORDER | TAP | TAP2 | TAP3 | Polynomial |
|---|---:|---:|---:|---:|---|
| PRBS7 | 7 | 6 | 0 | 0 | x^7 + x^6 + 1 |
| PRBS9 | 9 | 5 | 0 | 0 | x^9 + x^5 + 1 |
| PRBS13 | 13 | 12 | 2 | 1 | x^13 + x^12 + x^2 + x + 1 |
| PRBS15 | 15 | 14 | 0 | 0 | x^15 + x^14 + 1 |
| PRBS23 | 23 | 18 | 0 | 0 | x^23 + x^18 + 1 |
| PRBS31 | 31 | 28 | 0 | 0 | x^31 + x^28 + 1 |

The checker and stimulus generator must use the same bit order and advance
convention.

With the default `AUTO_ALIGN=1`, the checker captures the first `ORDER`
received bits, reconstructs the LFSR phase, and starts predicting later
bits. This assumes those acquisition bits are valid. `SEED` is ignored in
this mode. It performs acquisition once after reset; it does not
automatically re-acquire after a later burst of errors.

With `AUTO_ALIGN=0`, checking starts from `SEED`; the first expected bit
after reset is `SEED[ORDER-1]`.

## Typical PRBS7 parameters

```text
ORDER=7
TAP=6
SEED=127
AUTO_ALIGN=1
VTH_CLK=0.4
VTH_DATA=0.4
VTH_RST=0.4
SKIP_BITS=20
COUNT_SCALE=1u
REPORT_EVERY=100000
```

For a 0.8 V interface, the 0.4 V thresholds above are a useful starting
point. Place the rising sampling edge near the eye center. `SKIP_BITS`
ignores additional bits after automatic acquisition (or after reset in
fixed-seed mode) while still advancing the expected LFSR.

Set `INVERT_DATA=1` when the observed data polarity is inverted. Set
`STOP_ON_ERROR=1` for debugging when the simulation should stop on the
first mismatch.

## PRBS13 parameters

```text
ORDER=13
TAP=12
TAP2=2
TAP3=1
SEED=8191
AUTO_ALIGN=1
```

This implements `x^13 + x^12 + x^2 + x + 1`, with an 8191-bit period.

## Automatic sampling-clock alignment

Use `prbs_checker_auto.va` when aligning an external bit-rate clock to the
data is inconvenient. Its pins are:

```text
din rst err locked bit_count error_count
```

There is no `clk` pin. Set `BIT_RATE` to the binary serial data rate. The
checker detects `din` threshold crossings, starts sampling at
`SAMPLE_PHASE * UI` after a crossing, and continues at one sample/UI.
Later data crossings provide phase tracking.

Typical 56 Gb/s IEEE/OIF PRBS13 settings:

```text
ORDER=13
TAP=12
TAP2=2
TAP3=1
BIT_RATE=56e9
SAMPLE_PHASE=0.5
PHASE_GAIN=0.25
VTH_DATA=0.4
VTH_RST=0.4
LOCK_BITS=64
UNLOCK_ERRORS=3
COUNT_SCALE=1u
```

`locked` is asserted only after `LOCK_BITS` consecutive correct
predictions. Before lock, a mismatch triggers another PRBS phase
acquisition. After lock, `UNLOCK_ERRORS` consecutive errors drop lock and
start acquisition again.

The module still needs the correct `BIT_RATE`; a PRBS data stream alone
does not contain a transition at every UI, so its bit rate cannot be
unambiguously inferred from data edges.

### Cadence vsource PN13

Cadence `vsource` with `LFSR Mode=PN13` uses taps `13,4,3,1,0`, corresponding
to:

```text
x^13 + x^4 + x^3 + x + 1
```

Cadence numbers these taps from left to right starting at zero. Use the
non-leading, non-zero taps directly:

```text
ORDER=13
TAP=4
TAP2=3
TAP3=1
```

These are now the default tap values in `prbs_checker_auto.va`. Its default
`BIT_RATE` is also set to `32e9` for this test case.

