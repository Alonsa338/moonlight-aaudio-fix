**[English](README.md) | [Русский](README.ru.md)**

# Moonlight AAudio Low-Latency Fix (Google TV Streamer / Android TV 14)

A build of **Moonlight v12.2** with a native **AAudio low-latency** audio renderer.

This fixes the well-known issue of **audio delay (~0.5–1s)** on devices where
Android denies AudioTrack the fast (FAST) path and silently routes audio to a
deep `DEEP_BUFFER` — most notably **Google TV Streamer**, plus some other
Android TV boxes/TVs on Android TV 14.

---

## TL;DR

- **Download the APK** from the [Releases](../../releases) section and install it.
- The fix works automatically; on Android 8+ you can now manage it:
  in **Audio Settings** there is the **"Enable AAudio low-latency renderer"**
  toggle (on by default).
- If audio is still delayed, make sure in Moonlight settings that:
  - **"Enable audio FX"** is OFF (AAudio only works without audio effects);
  - **Stereo (2.0)** is selected, not 5.1/7.1 (the fix covers stereo only);
  - **"Enable AAudio low-latency renderer"** is ON.

---

## About the problem

Original ticket: [moonlight-stream/moonlight-android#1423](https://github.com/moonlight-stream/moonlight-android/issues/1423)

Known bug: on Google TV Streamer (Android TV 14) audio lags behind video by
~0.5–1s, while video and input respond normally. The issue appears not only in
Moonlight but also in Steam Link, GeForce Now, etc. The cause is at the
firmware/ROM level: Android uses a very long audio buffer for the HDMI output
and ignores the requested FAST path, routing audio into `DEEP_BUFFER` instead.

This is not a Moonlight bug — it is a quirk of specific devices.

---

## How this build fixes it

Moonlight was extended with a **native AAudio renderer** (C, JNI):

- On Android O+ (API 26+), and **only when system audio effects are disabled**,
  the **AAudio MMAP** path is used (exclusive, LOW_LATENCY) — it provides
  minimal latency.
- If AAudio is unavailable or fails to open, it automatically falls back to the
  standard AudioTrack path.
- A native ring buffer keeps latency from growing.
- Buffer sizes are optimized for minimal latency.
- A new **"Enable AAudio low-latency renderer"** setting was added in Audio
  Settings — you can turn AAudio off if a device misbehaves.
- Fixed a reconnect bug (after exiting a stream and reopening it, audio could go
  silent) — added a small delay when recreating the AAudio stream.

> ⚠️ **Known limitation:** the fix works for **stereo (2.0)** only. Surround
> modes (5.1/7.1) are not covered — with surround, the delay may return.

---

## Troubleshooting (FAQ)

### 🔊 Audio plays on both the PC and the TV at the same time

The Moonlight **"Play audio on PC"** setting only works with the NVIDIA stack
(GeForce Experience / Shield) and **does not affect Sunshine**. If during
streaming sound is heard on both the host PC and the client, that is normal
Sunshine behavior, not a bug of this build. To silence the PC:

1. Open the Sunshine Web UI: `http://localhost:47990`
2. Go to **Config → Audio/Video**
3. Set **Virtual Sink = Steam Streaming Speakers** (a silent virtual device),
   and leave **Audio Sink** empty.
4. Apply the settings and restart Sunshine.

If "Steam Streaming Speakers" is not in your Windows device list, enable the
**Install Steam audio drivers** option in Sunshine (or run Steam Remote Play
once so the driver gets installed).

### 🔁 No audio after reopening a stream

Update to **v2** of this build (it adds a delay when recreating the AAudio
stream). If the problem persists, turn off **"Enable AAudio low-latency
renderer"** in audio settings.

### 🔇 No audio at all

Check Moonlight settings: **"Enable audio FX"** is OFF, audio is set to
**Stereo (2.0)**, and **"Enable AAudio low-latency renderer"** is ON.

---

## Files

| File | Description |
|---|---|
| `Moonlight-12.2-AAudioFix-release.apk` | Ready-to-install signed build |
| `aaudio-fix.patch` | Patch against Moonlight v12.2 sources |

---

## Credits

- **Base:** official [moonlight-stream/moonlight-android](https://github.com/moonlight-stream/moonlight-android) v12.2
- **AAudio fix:** [@olibols](https://github.com/olibols), PR [ClassicOldSong#567](https://github.com/ClassicOldSong/moonlight-android/pull/567) (commit `fa0a309`)
- **Build/packaging:** this repository

## License

This project is distributed under **GPLv3** (same as Moonlight).
Please keep the attribution links.

---

## Build it yourself

1. `git clone --depth 1 https://github.com/moonlight-stream/moonlight-android.git`
2. `cd moonlight-android`
3. `git submodule update --init --recursive`
4. Apply the patch: `git apply aaudio-fix.patch`
5. Build: `./gradlew :app:assembleNonRootRelease`
   (requires Android SDK: platform 37, build-tools, NDK 29.0.14206865)
