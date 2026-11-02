# [T-7] Symphonia decoder adapter

Milestone: M0 · Type: feature · Layer: infra · Size: large · Status: todo

## Goal

Decode the supported input formats into `AudioBuffer`.

## Scope

- `AudioDecoder` in `infra-audio` with symphonia: WAV, FLAC, MP3, OGG/Vorbis, AAC (M4A).
- Integer PCM scaled to `f32` full scale; float samples passed through unchanged.
- Small fixtures committed, made once with ffmpeg from a known signal; the commands are stored next to them.

## Out of scope

- Resampling, channel conversion, Opus, HE-AAC.

## Acceptance criteria

- [ ] WAV (16, 24, 32f) and FLAC (16, 24) fixtures decode sample-exact.
- [ ] MP3, OGG and AAC fixtures match the source signal within tolerances set in the test plan.
- [ ] Mono and stereo inputs keep their channel count and sample rate.
- [ ] Unsupported or corrupt input returns the matching domain error and never panics.

## Test plan

_Filled when the task moves to `planned`._

## Implementation plan

_Filled when the task moves to `planned`._

## Notes

- Large: branch `t-7-symphonia-decoder` from `m0-foundation`, PR back into it.
- Gapless handling (MP3/AAC encoder delay) is decided in the plan.
