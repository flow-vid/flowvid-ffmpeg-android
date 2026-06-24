# flowvid-media3-ffmpeg

Builds an **LGPL** FFmpeg **audio** decoder extension for AndroidX Media3 (ExoPlayer), as a
self-contained AAR that FlowVidTV consumes. This is what lets ExoPlayer **decode** Dolby
(AC-3 / E-AC-3 / DD+) + DTS + TrueHD instead of relying on the TV's broken passthrough.

## Why this repo exists (and why NOT NextLib)
- **NextLib is GPLv3** → can't ship in closed-source FlowVid.
- NextLib also ships *shared* `libav*.so` that **collide with mpv's FFmpeg**, and its decoder can't
  resolve mpv's symbols (`swr_free` UnsatisfiedLinkError) — verified failure on-device.
- This repo uses the **official androidx/media `decoder_ffmpeg` module (Apache-2.0 wrapper)** and
  builds FFmpeg **statically linked** into a single `libffmpegJNI.so` (NO shared `libav*.so`), so it
  **does not collide with mpv** and has no external symbol dependency. Built **without `--enable-gpl`
  → LGPL**, scoped to the audio decoders we need.

## License posture (closed-source safe)
- Wrapper: **Apache-2.0** (androidx/media). FFmpeg: **LGPL** (no `--enable-gpl`; the AC-3/E-AC-3/DTS/
  TrueHD decoders are all in the LGPL set). → usable in closed-source FlowVid with LGPL compliance
  (dynamic `.so`, in-app notice, offer this repo as the source, don't checksum-pin the `.so`).
- **Patent note:** AC-3 (expired 2017) and E-AC-3/DD+ (expired ~Jan 2026) are low-risk to decode.
  DTS / AC-4 / TrueHD are still patented — keep those on passthrough; we only *enable* what we intend
  to use (default: `ac3 eac3`; `dca mlp truehd` are listed but consider leaving them disabled until a
  patent/licensing decision). See FlowVid `memory/flowvid-licensing.md`.

## How it works
GitHub Actions (hosted Ubuntu runner + NDK) on a `v*` tag:
1. clones `androidx/media` at the tag matching FlowVidTV's media3 version,
2. clones FFmpeg at the version that media3 tag expects,
3. builds FFmpeg for arm64-v8a (+armeabi-v7a) with only the audio decoders, **no `--enable-gpl`**,
4. assembles `lib-decoder-ffmpeg-release.aar`,
5. publishes it as a **GitHub Release asset** (no external service — all GitHub).

**Consuming it in FlowVidTV (simplest, no Maven/JitPack/R2):** download the `.aar` from this repo's
Release, drop it in `FlowVidTV/app/libs/flowvid-media3-ffmpeg.aar`, and uncomment in
`app/build.gradle.kts`:
```kotlin
implementation(files("libs/flowvid-media3-ffmpeg.aar"))
```
FlowVidTV already uses `EXTENSION_RENDERER_MODE_PREFER`, so `androidx.media3.decoder.ffmpeg.FfmpegAudioRenderer` is auto-detected from the AAR. Because the build is **audio-only**, PREFER never touches video. You only re-copy the AAR when you rebuild the lib (rare — i.e. when bumping media3). (Want zero manual copying? JitPack can auto-serve it, but native builds on JitPack are finicky — the Release→libs/ route is the reliable one.)

## To cut a build
1. Confirm `MEDIA3_TAG` in the workflow matches FlowVidTV's media3 version (currently `1.10.1`).
2. Confirm `FFMPEG_TAG` matches what `libraries/decoder_ffmpeg/README.md` of that media3 tag specifies.
3. `git tag v1.10.1-1 && git push --tags` → Actions builds + publishes the AAR.
4. In FlowVidTV, point the dependency at the published AAR and rebuild.

> First CI run may need a small tweak (NDK version / FFmpeg tag / build_ffmpeg args) — these native
> builds always settle on run 1-2. The workflow logs will say exactly what.
