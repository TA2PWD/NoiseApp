# [T-9] FLAC encoder adapter

Milestone: M0 · Type: feature · Layer: infra · Size: small · Status: todo

## Goal

Write an `AudioBuffer` as FLAC.

## Scope

- `AudioEncoder` with flacenc: 16-bit, 24-bit.
- Same saturation rule as T-8.

## Out of scope

- Clip policy, dither, compression level tuning.

## Acceptance criteria

- [ ] Decoding the output gives exactly the quantised input samples (lossless).
- [ ] Samples beyond ±1.0 come back at full scale.
- [ ] Stream info carries the right rate, channel count and depth.
- [ ] Works with a file sink and an in-memory sink.

## Test plan

_Filled when the task moves to `planned`._

## Implementation plan

_Filled when the task moves to `planned`._

## Notes

None.
