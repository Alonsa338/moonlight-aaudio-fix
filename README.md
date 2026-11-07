# Moonlight AAudio Low-Latency Fix (Google TV Streamer / Android TV 14)

Сборка **Moonlight v12.2** с нативным **AAudio low-latency** аудио-рендерером.

Это решает известную проблему **задержки звука (~0.5–1 сек)** на устройствах, где
Android отказывает AudioTrack в быстром (FAST) пути и молча отправляет звук в
глубокий буфер `DEEP_BUFFER` — прежде всего **Google TV Streamer**, а также
некоторые другие Android-приставки/ТВ на Android TV 14.

---

## TL;DR

- **Скачай APK** в разделе [Releases](../../releases) и установи его.
- **Ничего специально включать не нужно** — фикс работает автоматически.
  Он активируется при стандартных настройках (аудиоэффекты выключены, стерео).
- Если звук всё ещё отстаёт — убедись, что в настройках Moonlight:
  - **«Enable audio FX» выключен** (AAudio работает только без аудиоэффектов);
  - выбран **Stereo (2.0)**, а не 5.1/7.1 (фикс покрывает только стерео).

---

## О проблеме

Оригинальный тикет: [moonlight-stream/moonlight-android#1423](https://github.com/moonlight-stream/moonlight-android/issues/1423)

Известный баг: на Google TV Streamer (Android TV 14) звук отстаёт от картинки на
~0.5–1 секунду, при этом видео и ввод откликаются нормально. Проблема проявляется
не только в Moonlight, но и в Steam Link, GeForce Now и т.д. Причина — на уровне
прошивки/ROM: Android использует очень длинный аудио-буфер для выхода HDMI и
игнорирует запрошенный FAST-путь, перенаправляя звук в `DEEP_BUFFER`.

Это не баг самого Moonlight, а особенность конкретных устройств.

---

## Как это исправляет сборка

В Moonlight добавлен **нативный AAudio-рендерер** (C, JNI):

- На Android O+ (API 26+) и **только когда выключены системные аудиоэффекты**
  используется **AAudio MNAP**-путь (exclusive, LOW_LATENCY) — он даёт
  минимальную задержку.
- Если AAudio недоступен или не смог открыться — автоматический откат (fallback)
  на стандартный AudioTrack.
- Кольцевой буфер на нативной стороне не даёт задержке расти.

> ⚠️ **Известное ограничение:** фикс работает для **стерео (2.0)**. Режимы
> surround (5.1/7.1) не покрываются — при выборе surround задержка может
> вернуться.

---

## Файлы

| Файл | Описание |
|---|---|
| `Moonlight-12.2-AAudioFix-release.apk` | Готовая подписанная сборка для установки |
| `aaudio-fix.patch` | Патч к исходникам Moonlight v12.2 |

---

## Авторство

- **Основа:** официальный [moonlight-stream/moonlight-android](https://github.com/moonlight-stream/moonlight-android) v12.2
- **AAudio-фикс:** [@olibols](https://github.com/olibols), PR [ClassicOldSong#567](https://github.com/ClassicOldSong/moonlight-android/pull/567) (коммит `fa0a309`)
- **Сборка/упаковка:** этот репозиторий

## Лицензия

Проект распространяется под **GPLv3** (как и Moonlight).
Пожалуйста, сохраняйте ссылки на авторство.

---

## Как собрать самому

1. `git clone --depth 1 https://github.com/moonlight-stream/moonlight-android.git`
2. `cd moonlight-android`
3. `git submodule update --init --recursive`
4. Примени патч: `git apply aaudio-fix.patch`
5. Собери: `./gradlew :app:assembleNonRootRelease`
   (нужны Android SDK: platform 37, build-tools, NDK 29.0.14206865)
