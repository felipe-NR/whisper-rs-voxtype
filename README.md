# whisper-rs-voxtype

[whisper-rs](https://github.com/tazz4843/whisper-rs) 0.16.0 and whisper-rs-sys
0.15.0 (bundled whisper.cpp 1.8.3), as published on crates.io, plus a runtime
encoder callback for whisper.cpp. Used by the `feat/xdna-npu-encoder` branch of
[felipe-NR/voxtype](https://github.com/felipe-NR/voxtype) to run the Whisper
encoder on an AMD XDNA NPU, through `[patch.crates-io]`.

This is a temporary carrier, not a maintained fork. The goal is to upstream the
callback to ggml-org/whisper.cpp and whisper-rs.

## Changes on top of the crates.io sources

- `whisper_set_encoder_callback` / `whisper_set_encoder_callback_with_state`:
  an external encoder gets each window's log-mel (n_mel x 2*n_ctx, row-major)
  and may write the encoder output after `ln_post` (n_ctx x n_state). It
  returns 0 when it wrote the output (the ggml conv and encoder graphs are
  skipped), 1 to decline the window (whisper.cpp runs its own encoder), or a
  negative value on failure. Cross-attention K/V, decoding, language detection
  and timestamps stay in whisper.cpp.
- `whisper_full` applies `audio_ctx` before language auto-detection, so
  detection encodes the same window as the transcription and an external
  encoder can serve both from one encode.
- `whisper_set_audio_ctx_with_state`, for callers that run language detection
  themselves.
- whisper-rs: `WhisperState::set_encoder_callback`, `WhisperState::set_audio_ctx`
  and the exported `EncoderCallback` type.

See the commit history for each change on its own.

## License

MIT, see `LICENSE`. The upstream code keeps its own notices:
whisper-rs and whisper-rs-sys were released under the Unlicense
(`whisper-rs/LICENSE`), which allows relicensing; whisper.cpp and ggml are MIT
(`whisper-rs-sys/whisper.cpp/LICENSE`), and that notice stays with them.
