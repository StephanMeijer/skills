# Image Conversion

## Choose the Path

Prefer ImageMagick (`magick`) for still images. Use FFmpeg when ImageMagick is absent, for image sequences, or when the installed FFmpeg build has the required encoder. Use dedicated tools such as `cwebp`, `dwebp`, `avifenc`, or `pngquant` only after checking their supported color, alpha, animation, and metadata behavior.

Inspect with one of:

```sh
identify -verbose "input.png"
ffprobe -v error -show_streams -show_format -of json "input.png"
file "input.png"
```

Check the selected encoder directly. For FFmpeg, use `ffmpeg -hide_banner -encoders` and `ffmpeg -hide_banner -h encoder=NAME` rather than assuming a distribution enables a codec.

## Preserve Meaningful Properties

- Preserve pixel dimensions unless resizing is requested.
- Preserve alpha only in formats that support it. JPEG cannot represent transparency; obtain a background color before flattening.
- Preserve ICC profiles, EXIF orientation, bit depth, and animation when they matter. Verify because metadata defaults vary by format and tool.
- Apply orientation before deliberately removing orientation metadata.
- Avoid lossy-to-lossy conversion when the original or a lossless source is available.
- State when CMYK, HDR, wide-gamut color, layers, or vector content will be flattened or transformed.

## Command Patterns

Perform a direct still-image conversion with ImageMagick:

```sh
magick "input.png" "output.webp"
```

Set an explicit lossy quality only when the contract permits it:

```sh
magick "input.jpg" -quality 85 "output.webp"
```

Use FFmpeg as a still-image fallback and prevent accidental sequence output:

```sh
ffmpeg -hide_banner -i "input.png" -frames:v 1 "output.webp"
```

For resizing, preserve aspect ratio and avoid upscaling unless requested. With ImageMagick, `1920x1920>` constrains both dimensions and the trailing `>` prevents enlargement:

```sh
magick "input.png" -resize '1920x1920>' "output.webp"
```

Do not add `-strip` by default; it intentionally removes profiles and metadata. Do not rely on a filename extension alone to select an encoder when a tool provides an explicit format option.

## Validate

Identify the output with a second tool when possible. Confirm format, dimensions, alpha, frame count, bit depth, orientation, color space or profile, and expected metadata. Decode every frame of animated output. Render the image over light and dark backgrounds when transparency is important, and compare a representative crop at native resolution for compression artifacts.
