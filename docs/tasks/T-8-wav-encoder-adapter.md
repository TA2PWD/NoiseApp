# [T-8] WAV encoder adapter

Milestone: M0 · Type: feature · Layer: infra · Size: small · Status: todo

## Goal

Write an `AudioBuffer` as WAV.

## Scope

- `AudioEncoder` with hound: 16-bit, 24-bit, 32-bit float.
- Integer depths saturate out-of-range samples, never wrap. Clip policy and dither stay open (PLAN, M3).

## Out of scope

- FLAC (T-9), clip policy, dither, zip.

## Acceptance criteria

- [ ] Round trip of generated sine and noise: 32f bit-exact; 16/24-bit within one quantisation step.
- [ ] Samples beyond ±1.0 come back at full scale, not wrapped.
- [ ] Header carries the right rate, channel count and depth.
- [ ] Works with a file sink and an in-memory sink.

## Test plan

_Filled when the task moves to `planned`._

## Implementation plan

_Filled when the task moves to `planned`._

## Notes

None.
