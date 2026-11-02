# NoiseApp Plan

## Goal

Split a music file into stems and export each stem as its own file.
The stems are then used for sampling: cutting regions, cleaning noise.

## Scope

v1 (M0-M7):
- Input: any format Symphonia decodes (WAV, FLAC, MP3, OGG/Vorbis, AAC/M4A).
- Models: `htdemucs` (4 stems) and `htdemucs_6s` (6 stems), user-selectable.
- Inference: ONNX Runtime, CUDA when available, CPU fallback.
- Output: WAV (16 / 24 / 32-bit float), FLAC (16 / 24-bit).
- Separation result saved as a project so later edits do not re-run the model.
- Region export: pick start/end, export that region from selected stems.
- Interfaces: two executables, `noiseapp` (CLI) and `noiseapp-web` (local web UI).
- Export delivery: browser download (single file or zip), optionally also
  written to a server-side output folder.
- Packaging: release archive per OS (Windows, Linux) with both executables and
  the ONNX Runtime CUDA provider libraries; Docker CPU image, Docker CUDA image.

Later (M8+): noise reduction, normalize, fade in/out, `htdemucs_ft`.

Non-goals: real-time separation, multitrack editing/mixing, native GUI,
multi-user hosting, authentication, macOS native build.

## Architecture

See CLAUDE.md for crate list and dependency rule.

### Domain (entities, value objects)

- `SampleRate`, `ChannelCount`
- `AudioBuffer`: planar f32 samples + sample rate; invariant: all channels same length.
- `StemKind`: Vocals, Drums, Bass, Other, Guitar, Piano.
- `ModelSpec`: id, ordered stems, sample rate, segment length, overlap, member
  networks with per-stem weights (bag of models, e.g. `htdemucs_ft`).
- `StemSet`: one `AudioBuffer` per `StemKind`; invariant: equal length and rate.
- `Region`: start/end in samples; invariant: start < end <= length.
- `ExportFormat`: `Wav(WavDepth)` | `Flac(FlacDepth)`.
- `Project`: source info, model id + SHA256, separation params, stem file references.

### Domain ports (traits)

- `AudioDecoder`      path -> AudioBuffer
- `AudioEncoder`      AudioBuffer + ExportFormat -> byte sink (file or zip entry)
- `Resampler`         AudioBuffer + target rate -> AudioBuffer
- `StemSeparator`     AudioBuffer + ModelSpec + progress + cancel -> StemSet
- `ModelStore`        locate and verify model files
- `ProjectRepository` save / load / list / delete Project and its stems
- `Progress`, `CancelToken`

### Application (use cases)

- `SeparateTrack`: decode -> stereo (as Demucs: mono duplicated, first two of
  more channels) -> resample to model rate -> separate -> save project
- `ExportStems`: project + selected stems + optional region + format -> files
- `ListModels`, `ListProjects`, `DeleteProject`

### Infrastructure

- infra-audio: symphonia (decode), hound (WAV), flacenc (FLAC), rubato (resample);
  integer encoders saturate out-of-range samples, never wrap
- infra-demucs: ort + realfft; mirrors Demucs `apply_model` with `shifts=0`:
  normalize -> segment (overlap 0.25, triangular weights) -> STFT ->
  ONNX -> iSTFT -> overlap-add -> denormalize
- infra-storage: `<name>.noiseproj/` = 32-bit float WAV per stem + `manifest.json`
  (`schema_version`, app version, model id + SHA256, separation params)

### Presentation

- web (`noiseapp-web`): axum, REST + SSE for job progress. Separation jobs run
  one at a time on `spawn_blocking`; the job queue lives in the web layer.
  Binds `127.0.0.1` by default; `NOISEAPP_BIND` overrides it (Docker binds
  `0.0.0.0` inside; publish as `-p 127.0.0.1:<port>:<port>`). Data directory
  overridable with `NOISEAPP_DATA_DIR` (Docker volume). No auth, so: Host and
  Origin allowlist; mutating endpoints require JSON or a custom header (no
  cross-site form posts); output folder only from server config; upload and
  zip file names sanitised against path traversal.
- frontend: Svelte + TypeScript (Vite), wavesurfer.js with the regions
  plugin. Playback via Web Audio with per-stem solo and mute.
- theme: `frontend/src/theme/tokens.json` (derived from `frontend/DESIGN.md`)
  is the single token source; `tokens.css` (custom properties) and
  `tokens.ts` (typed constants for wavesurfer and canvas code) are
  generated from it. No hard-coded design values anywhere else.
- cli (`noiseapp`): same use cases for scripting.

### Model export (dev-time only)

`tools/export/` Python script (pinned `demucs`, `torch`, `onnx` versions):
1. Loads `htdemucs` / `htdemucs_6s` from the `demucs` package.
2. Exports the network to ONNX (pinned opset, fixed segment shape) with
   STFT/iSTFT moved outside the graph; checks it against PyTorch in onnxruntime.
3. Writes golden data with `shifts=0`: CC0 input clips (several segments with a
   partial last one; shorter than one segment; mono), torch STFT output, one
   segment's ONNX input/output, PyTorch stem outputs.

The app never runs Python.

### Model distribution

- ONNX files and golden data are GitHub Release assets (tag `models-v1`) of
  this public repo (no token needed), shipped with the Demucs licence notice.
- `ModelStore` downloads them on demand and verifies them against SHA256
  values built into the app.
- CI downloads the same assets for the golden test.

## Testing strategy

- domain: unit tests for invariants.
- application: use case tests with in-memory fake ports.
- infra-audio: round-trip tests on generated signals (sine, noise); decode
  tests on small committed fixtures (MP3, OGG, AAC made once with ffmpeg).
- infra-demucs: STFT/iSTFT and single-segment ONNX parity vs the torch
  fixtures (round-trip alone misses window, padding and scaling errors);
  golden test vs PyTorch output. The golden test passes when, for every stem,
  max |rust - pytorch| < 1e-3 AND SDR(rust, pytorch) > 60 dB, where
  SDR = 10 * log10(sum(ref^2) / sum((ref - est)^2)); stems with reference
  RMS below -40 dBFS use the max-diff check only.
- web: API integration tests with fake ports (`tower::ServiceExt::oneshot`).
- frontend: unit tests with Vitest; token generator tests (JSON -> CSS/TS
  output); stylelint + custom check rejecting raw design values outside
  the token files, run in CI.
- cli: smoke tests of command parsing.
- CI: GitHub Actions on windows-latest + ubuntu-latest, CPU only.

## Milestones

| ID | Name          | Outcome |
|----|---------------|---------|
| M0 | Foundation    | Workspace, CI, domain core types, decode (WAV, FLAC, MP3, OGG, AAC), WAV/FLAC encode |
| M1 | Model export  | ONNX files + golden data (incl. STFT and single-segment tensors) for both models |
| M2 | DSP           | Resampler adapter, STFT/iSTFT matching the torch fixtures |
| M3 | Separation    | CPU inference passes golden test; `SeparateTrack`; project storage; `noiseapp separate` |
| M4 | Runtime       | CUDA, progress, cancel, model download + SHA256 check |
| M5 | Web UI        | Theme tokens + token lint first; then API + SSE progress, upload, stem list, solo/mute playback, download/zip |
| M6 | Regions       | Waveform, region select, region export via `ExportStems` |
| M7 | Packaging     | Release archives (both executables, embedded frontend, CUDA provider libs), Docker CPU + CUDA images, release CI |
| M8 | Processing    | Noise reduction, normalize, fade |

## Risks

- HTDemucs transformer export to ONNX (reference: sevagh/demucs.onnx).
- Numeric parity with PyTorch across CPU and CUDA.
- CUDA 12 + cuDNN 9 setup on Windows for ONNX Runtime.
- CUDA Docker image size and host driver compatibility.

## Open questions

- M0: licence for the public repo?
- M3: export sample rate: model rate (44.1 kHz) or source rate?
- M3: stems of a mono source: export mono or stereo?
- M3: integer export: clip policy (rescale / clamp / none), 16-bit dither?
- M4: CUDA parity: local-only test with its own threshold, or TF32 off?
- M4: crate for the `ModelStore` adapter; HTTP client.
- M5: default data directory; cleanup of old projects.
- M5: upload limits (file size, duration).
- M5: does "fonts follow DESIGN.md" cover only the family, or also case,
  letter-spacing and weight?
- M5/M6: browser preview format of stems; server-side waveform peaks.
- M7: CUDA 12 + cuDNN 9 runtime: bundled in the archive or a prerequisite?
