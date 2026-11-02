# [T-4] Domain audio types

Milestone: M0 · Type: feature · Layer: domain · Size: small · Status: todo

## Goal

Core audio value types for every later layer.

## Scope

- `SampleRate`: Hz, greater than zero.
- `ChannelCount`: at least one.
- `AudioBuffer`: planar `f32` samples + `SampleRate`; all channels the same length; exposes channel count, frame count and duration.
- `thiserror` errors for invalid input.

## Out of scope

- Channel conversion, resampling, I/O.

## Acceptance criteria

- [ ] Rate 0, zero channels and channels of unequal length are rejected with a typed error.
- [ ] A valid buffer reports the right channel count, frame count, sample rate and duration.

## Test plan

_Filled when the task moves to `planned`._

## Implementation plan

_Filled when the task moves to `planned`._

## Notes

None.
