# Video Conversion

## Inspect and Decide

Inspect every stream, disposition, chapter, and relevant tag:

```sh
ffprobe -v error -show_streams -show_chapters -show_format -of json "input.mkv"
```

Determine the target container, playback environment, codec compatibility, maximum dimensions or size, HDR or SDR behavior, alpha needs, subtitle handling, and required audio tracks. Query `ffmpeg -hide_banner -encoders`, `ffmpeg -hide_banner -filters`, and the selected encoder's help before constructing the command.

Prefer remuxing when all required streams are compatible with the target container. Transcode only the incompatible streams. Never silently drop secondary audio, subtitles, chapters, cover art, or accessibility tracks.

## Command Patterns

Remux selected streams without quality loss:

```sh
ffmpeg -hide_banner -i "input.mkv" -map 0:v:0 -map '0:a?' -map '0:s?' -map_metadata 0 -map_chapters 0 -c copy "output.mkv"
```

Use explicit, compatibility-oriented H.264/AAC MP4 settings when that target is requested:

```sh
ffmpeg -hide_banner -i "input.mov" -map 0:v:0 -map '0:a?' -map_metadata 0 -map_chapters 0 -c:v libx264 -preset medium -crf 20 -pix_fmt yuv420p -c:a aac -b:a 192k -movflags +faststart "output.mp4"
```

Use an even-dimension scale only when odd dimensions make the chosen codec fail, and preserve aspect ratio:

```sh
-vf 'scale=trunc(iw/2)*2:trunc(ih/2)*2'
```

Treat CRF, preset, bitrate, pixel format, and codec choices as target-dependent. Check hardware encoders before use; they often trade efficiency or portability for speed. Preserve HDR only with an end-to-end HDR-capable codec, pixel format, color metadata, filter chain, and container. Do not accidentally tone-map or discard HDR metadata.

When burning subtitles, confirm font availability and accept that the result is no longer toggleable. When retaining subtitle streams, choose a subtitle codec supported by the target container.

## Animation

Treat GIF, APNG, and animated WebP as timed media. Preserve loop count, timing, transparency, and frame count when the target supports them. GIF has severe palette and alpha limits; generate and use a deliberate palette rather than accepting uncontrolled banding when visual quality matters.

## Validate

Probe the output and compare container, codecs, duration, dimensions, display aspect ratio, frame rate, time base, pixel format, color properties, stream count and order, dispositions, chapters, rotation, and metadata. Decode the complete artifact:

```sh
ffmpeg -v error -i "output.mp4" -f null -
```

Play the beginning, representative motion and seek points, and the end. Check audio/video synchronization, orientation, subtitles, scene changes, black levels, color, dropped or duplicated frames, and seeking. Test in the named target player or browser when compatibility is part of the request.
