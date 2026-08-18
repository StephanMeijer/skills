# Audio Conversion

## Inspect and Select

Inspect streams before choosing a codec:

```sh
ffprobe -v error -show_streams -show_format -of json "input.flac"
```

Determine whether the target needs lossless audio, broad playback support, streaming efficiency, editing headroom, or a constrained sample rate and channel layout. Check encoders with `ffmpeg -hide_banner -encoders` and inspect the chosen encoder with `ffmpeg -hide_banner -h encoder=NAME`.

Prefer:

- FLAC for lossless compressed archival audio;
- WAV with an explicit PCM format for editing or interchange when file size is acceptable;
- AAC in M4A for broad consumer-device support;
- Opus for efficient speech, streaming, and modern playback targets;
- MP3 only for compatibility requirements.

Never describe a lossy transcode as preserving quality. Avoid transcoding one lossy codec to another when the original source or a compatible stream copy is available.

## Command Patterns

Map the intended audio stream explicitly and discard video only when audio-only output is requested:

```sh
ffmpeg -hide_banner -i "input.mov" -map 0:a:0 -vn -map_metadata 0 -c:a flac "output.flac"
```

Common explicit targets include:

```sh
ffmpeg -hide_banner -i "input.flac" -map 0:a:0 -vn -map_metadata 0 -c:a libopus -b:a 128k "output.opus"
ffmpeg -hide_banner -i "input.flac" -map 0:a:0 -vn -map_metadata 0 -c:a aac -b:a 192k "output.m4a"
ffmpeg -hide_banner -i "input.flac" -map 0:a:0 -vn -map_metadata 0 -c:a libmp3lame -q:a 2 "output.mp3"
ffmpeg -hide_banner -i "input.flac" -map 0:a:0 -vn -map_metadata 0 -c:a pcm_s24le "output.wav"
```

Treat these values as starting points, not universal requirements. Preserve the source sample rate and channel layout unless the target contract requires a change. Apply loudness normalization only when requested; it changes the signal and should use measured, repeatable parameters.

For a container change with an already-compatible stream, use stream copy:

```sh
ffmpeg -hide_banner -i "input.mka" -map 0:a:0 -map_metadata 0 -c:a copy "output.m4a"
```

Confirm codec/container compatibility before relying on stream copy.

## Validate

Probe the output and compare codec, duration, sample rate, channel count and layout, bit depth when meaningful, bitrate or lossless status, tags, chapters, and cover art. Decode the complete file:

```sh
ffmpeg -v error -i "output.m4a" -f null -
```

Listen to the beginning, a representative middle section, and the end. Check for truncation, silence, clipping, channel swaps, clicks, or unexpected loudness changes. For speech, verify intelligibility; for gapless or looped sources, verify boundary behavior.
