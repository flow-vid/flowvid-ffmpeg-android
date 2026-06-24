# flowvid-media3-ffmpeg

LGPL FFmpeg audio decoder for AndroidX Media3 / ExoPlayer, packaged as an AAR.
Decodes **AC-3** and **E-AC-3** (Dolby Digital / Dolby Digital Plus) in software.

Built from the official [`androidx/media`](https://github.com/androidx/media) `decoder_ffmpeg`
module, with FFmpeg statically linked and configured **without `--enable-gpl`** (LGPL).

## Build

GitHub Actions builds and publishes the AAR on a version tag:

```sh
git tag v1.10.1-1 && git push --tags
```

The workflow clones `androidx/media` and FFmpeg at the pinned versions, builds the decoder, and
attaches `lib-decoder-ffmpeg-release.aar` to a GitHub Release.

| Setting     | Value             |
| ----------- | ----------------- |
| Media3 tag  | `1.10.1`          |
| FFmpeg      | `release/6.0`     |
| Decoders    | `ac3`, `eac3`     |
| ABIs        | arm64-v8a, armeabi-v7a, x86, x86_64 |
| NDK         | `27.2.12479018`   |

The Media3 tag must match the `androidx.media3` version used by the consuming app. The FFmpeg branch
is the one that Media3 tag's `libraries/decoder_ffmpeg/README.md` specifies.

## Usage

Download the `.aar` from the [Releases](../../releases), place it in the app's `libs/`, and add:

```kotlin
implementation(files("libs/flowvid-media3-ffmpeg.aar"))
```

With `EXTENSION_RENDERER_MODE_PREFER`, `androidx.media3.decoder.ffmpeg.FfmpegAudioRenderer` is picked
up automatically. The build is audio-only, so no video renderer is added.

## License

- Wrapper (`androidx/media`): Apache-2.0
- FFmpeg: LGPL (built with `--disable-gpl`; no x264/x265)

The AAR ships FFmpeg as a separate dynamically-linked library. The exact source is the `release/6.0`
branch of <https://git.ffmpeg.org/ffmpeg.git>; the build configuration is in
[`.github/workflows/build-and-publish.yml`](.github/workflows/build-and-publish.yml).
