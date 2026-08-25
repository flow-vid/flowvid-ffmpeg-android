# flowvid-media3-ffmpeg

LGPL FFmpeg audio decoder extension for AndroidX Media3, packaged as an Android AAR. The build enables
AC-3, E-AC-3, DTS (`dca`), MLP and TrueHD audio decoders. It does not enable video decoders or GPL
components.

FFmpeg's static libraries are linked into Media3's single `libffmpegJNI.so`. The AAR contains no
separate shared `libav*.so` files, so it does not collide with FlowVid's libmpv libraries.

## Reproducible release build

| Input | Pin |
| --- | --- |
| AndroidX Media3 | `5fb306449733dd71595700c1227ad6087578c559` (tag `1.10.1`) |
| FFmpeg | `3f92512fd1fd6f5e6d6eb45a156c352835314d69` (tested `release/6.0` commit) |
| Android NDK | `27.2.12479018` |
| Minimum SDK | API 26 |
| Decoders | ac3, eac3, dca, mlp, truehd |
| ABIs | arm64-v8a, armeabi-v7a, x86, x86_64 |

The tag-triggered workflow checks out those exact commits, builds all four ABIs, rejects unexpected
shared FFmpeg libraries, and publishes a versioned AAR with `build-manifest.txt`, `SHA256SUMS.txt`
and a GitHub build-provenance attestation. It fills a draft first and publishes only after every
asset is present and digest-checked. GitHub Actions dependencies are pinned to full commit SHAs.

After reviewing and committing the recipe, create a new immutable release tag:

```sh
git tag v1.10.1-5
git push origin main
git push origin v1.10.1-5
```

## Usage

Download the AAR from [Releases](../../releases), place it in the Android client's `libs` directory
as `flowvid-media3-ffmpeg.aar`, and reference it with Gradle:

```kotlin
implementation(files("libs/flowvid-media3-ffmpeg.aar"))
```

With Media3's extension renderer mode set to prefer extensions,
`androidx.media3.decoder.ffmpeg.FfmpegAudioRenderer` is selected for supported audio tracks.

## License

- Media3 decoder wrapper: Apache License 2.0.
- FFmpeg binary composition: GNU LGPL, built without `--enable-gpl` or nonfree components.
- Build scripts and repository documentation: MIT, see [LICENSE](LICENSE).

Exact source commits and build configuration are recorded in the release manifest and this
repository's workflow.
