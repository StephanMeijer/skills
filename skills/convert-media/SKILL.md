---
name: convert-media
description: Convert image, audio, and video files with local command-line tools while preserving requested quality, compatibility, metadata, streams, and dimensions. Use when asked to change a media format or container, transcode codecs, remux video, extract audio, make web-compatible media, resize during conversion, or batch-convert raster or animated media. Do not use for document, spreadsheet, presentation, PDF, or ebook conversion.
---

# Convert Media

Convert media with an installed CLI, preserve the source, and prove the result is usable. Choose settings from the user's playback, editing, size, quality, and compatibility requirements instead of treating a new file extension as sufficient.

## Establish the Conversion Contract

1. Identify every input and the exact output format, destination, and naming rule.
2. Ask only when an unresolved choice materially changes the result, such as lossy versus lossless output, target device, maximum size, transparency, animation, or required streams.
3. Inspect the source rather than trusting its extension. Prefer `ffprobe` for audio and video, ImageMagick's `identify` for still images, and `file` as a coarse fallback.
4. Record the source's format, codecs, duration, dimensions, frame rate, sample rate, channels, color or alpha characteristics, metadata, and embedded streams when relevant.
5. Keep the input unchanged. If the requested output path exists, do not overwrite it without explicit permission; select a distinct path or ask.

## Discover Capabilities

Probe commands with `command -v`; never assume a named tool or codec is installed. Inspect the selected tool's own help, format list, encoder list, or version before relying on optional features.

Prefer:

- FFmpeg and FFprobe for video, audio, animation, remuxing, and general fallback conversion;
- ImageMagick for still-image format, geometry, color-profile, and alpha-aware work;
- purpose-built lossless optimizers only when optimization, rather than format conversion, is requested.

Do not install packages or download binaries unless the user authorizes changing the environment. If no suitable converter exists, name the missing capability and stop before modifying any files.

## Route the Task

- Read [references/images.md](references/images.md) for still images, transparency, color profiles, or image sequences.
- Read [references/audio.md](references/audio.md) for audio-only conversion or audio extraction.
- Read [references/video.md](references/video.md) for video, animation, subtitles, stream mapping, remuxing, or video with audio.

For mixed requests, read each applicable reference. Treat animated GIF, APNG, and animated WebP as video-like timelines even when their filename looks like an image.

## Convert Deliberately

1. Decide whether the task is a lossless rewrap, lossless transcode, or lossy transcode. Remux compatible streams instead of re-encoding them.
2. Select codecs, pixel or sample formats, quality controls, and dimensions that meet the stated target. Do not upscale, change frame rate, discard alpha, flatten animation, reduce bit depth, or remove streams unless required or approved.
3. Make stream selection explicit for files with multiple audio tracks, subtitles, cover art, or attachments. Preserve relevant metadata by default; remove it only when requested for privacy or size.
4. Write to a temporary output on the destination filesystem. Promote it to the requested path only after validation succeeds.
5. For batches, test one representative file first, then apply the same decision rule to each input. Avoid shell word-splitting and preserve a deterministic input-to-output mapping.

## Verify the Artifact

Require all applicable checks:

1. Confirm the command exits successfully and the output is non-empty.
2. Probe the output independently and confirm its container, codec, duration, dimensions, streams, and metadata match the contract.
3. Decode the complete output with the chosen tool's error-only mode when practical; metadata inspection alone does not detect corrupt frames or samples.
4. Open, play, or render a representative portion through the real consumption surface when one is available. For a batch, inspect at least one output plus any edge cases.
5. Compare duration, aspect ratio, orientation, frame count, channel layout, transparency, and file size as applicable. Explain intentional differences.

Report the exact command, produced path, before-and-after properties, lossy choices, discarded information, and any validation surface that was unavailable.
