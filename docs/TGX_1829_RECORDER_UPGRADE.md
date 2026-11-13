# Telegram X Recorder 1.4.0 — обновление до Telegram X 1829

## Источники

- Исходный Recorder: 1.3.0 на Telegram X 0.29.1.1819 (`dev` = `1d20ced2`, tgcalls уже in-tree как subtree).
- Upstream: тег `v0.29.2.1829-beta` = `TGX-Android/Telegram-X:main` `7fe047c8861677845621922522c651bac6924d0c`, 123 коммита после 1819.
- Работа велась прямо в `dev`, без отдельных веток. `main`, теги и релизы не трогались.

## Коммиты

| Коммит | Что |
| --- | --- |
| `465c36d8` | Merge `v0.29.2.1829-beta` в Recorder |
| `7165ccdb` | `git subtree pull` tgcalls TGX production `97c1537a` |
| `cb017a2d` | Identity-тест ожидает 1.4.0 / 1829 |

## Разрешение конфликтов

- `app/jni/tgvoip/third_party/tgcalls`: upstream всё ещё хранит tgcalls как gitlink сабмодуля (`97c1537`). Оставлен наш subtree-каталог, gitlink отброшен, в `.gitmodules` нет записи tgcalls.
- `TGCallService`: upstream переписал foreground-логику звонков («Rework settings of calls», «Ask for the microphone when answering a call», «Fix incoming calls with the app closed», «Protect incoming calls by the passcode»). Взята версия upstream, поверх неё перенесён только API записи: `CallRecordingListener`, `start/pause/resume/stopCallRecording`, состояние/длительность записи и callback `onCallRecordingStateChanged` от tgcalls.
- `CallForegroundStateMachine` и его тест удалены вместе с recorder-проверкой `prepareForIncomingCallAnswer` в `CallManager.acceptCall`: это был собственный обход проблем FGS на Android 14, которые upstream теперь решает сам (микрофон запрашивается при ответе, служба переходит в microphone-foreground при видимом UI, есть периодическая проверка микрофона). `AndroidManifest` — как в upstream (`phoneCall|microphone|mediaPlayback`; действия звонков через `${applicationId}`).
- `CallController`: UI записи и listener сохранены рядом с новой upstream-обработкой пропущенного звонка под паролем и `TGCallService.checkIncomingNotification()`.
- `app/jni/CMakeLists.txt`: взята версия upstream (whitelist flavors удалён upstream).
- `app/build.gradle.kts`: идентичность Recorder (`ka.soft.tgxr`, release-версия) рядом с новой upstream-логикой google-services/Firebase; неиспользуемый `generateBaselineProfile` убран.

## Настройки записи

Upstream вынес настройки звонков в отдельный раздел **Настройки → Уведомления → Звонки** (`SettingsNotificationController`, `Section.CALLS`). Блок «Запись звонков» (автозапись, файлы записи, список записей) перенесён туда, после блока пароля для звонков. `SettingsThemeController` теперь совпадает с upstream.

## tgcalls

`git subtree pull --prefix=app/jni/tgvoip/third_party/tgcalls https://github.com/TGX-Android/tgcalls production` → `97c1537a` (TelegramMessenger/tgcalls@ea5153e + TGX Android platform), ровно то, что пинит 1829. Слияние без конфликтов. Проверено: дерево subtree = upstream `97c1537a` + те же 3 файла recorder-патча (+25/−5): `Instance.h` (`createAudioFrameProcessor`), `ThreadLocalObject.h` (`reset`), `v2/InstanceV2Impl.cpp` (подключение audio frame processor и reset при stop). Нативная часть Recorder (`CallRecorder.*`, `recorder/RecorderTgCallsAdapter.*`, `tgvoip.cpp`, `tgvoip/CMakeLists.txt`) не конфликтовала и компилируется с новым WebRTC.

## Зависимости

Все gitlinks совпадают с `v0.29.2.1829-beta` (`git submodule update --init --recursive`, без `--remote`):

| Компонент | SHA |
| --- | --- |
| tgcalls (subtree, upstream-часть) | `97c1537a63c819109f66fcfd65d685fb5d08763e` |
| WebRTC | `51b8a449a6` (ali-fareed/webrtc@590a8b2) |
| TDLib wrapper (prebuilt, LFS) | `6237ef3d91` (tdlib/td@c8828d5) |
| FFmpeg | `30e346c76f` |
| lz4 | `c72c7c0ec1` |
| tlottie (вместо rlottie) | `494e37e536` |
| libprisma (новый) | `d74f039056` |
| X-Annotation (новый) | `7750023b51` |
| X-Core / X-Android / tdlib-utils | `0dc5218145` / `746d52a389` / `68826290d4` |

TDLib приходит готовыми `libtdjni.so` через LFS — пересборка не нужна. Каталоги удалённых сабмодулей `rlottie` и `pixman` удалены из рабочей копии.

Новое требование upstream: **Rust 1.99.1** с Android-таргетами для tlottie (`version.rust`, `rust.dir` в `local.properties`). Установлен изолированно: официальный `rustup-init.exe` (SHA-256 сверен с static.rust-lang.org), `RUSTUP_HOME`/`CARGO_HOME` в `builds/tools/rust`, PATH и система не менялись; `rust.dir=<repo>/builds/tools/rust/rustup`.

Flavor `tens` переименован upstream в `nop` (Android 7–9); имя APK прежнее — `-nougatOreoPie`.

## Сборка

```powershell
$env:JAVA_HOME = 'C:/tools/java/jdk-25.0.4.1+1'
$env:MSYSTEM = 'UCRT64'
.\gradlew.bat :app:assembleLatestDebug :app:testLatestDebugUnitTest :app:assembleLatestRelease :app:assembleNopRelease :app:assembleQrsRelease -Pabi=arm --console=plain --max-workers=4
```

**BUILD SUCCESSFUL in 48m 8s**, 935 задач. Лог: `builds/tgx-1829-build.log`.

Unit tests: `CallRecordingRepositoryTest` 24, `ApplicationIdentityTest` 3 — 27 тестов, 0 ошибок/пропусков.

## APK

Проверка `builds/verify1829.py` (лог `builds/tgx-1829-apk-verification.log`, детали `builds/tgx-1829-verify/`): package `ka.soft.tgxr`, versionName/versionCode, обе ABI, JNI-экспорты записи в `libtgcallsjni.so` обеих ABI, DEX-члены Recorder, выравнивание 16 KiB, подпись; сертификат совпадает с Recorder 1.2.0/1.3.0 (`9d794556…cbd4`).

| Android | flavor | versionCode | versionName | Размер | SHA-256 |
| --- | --- | ---: | --- | ---: | --- |
| 13+ | latest debug | 1829000 | `1.4.0-arm-debug` | 126222153 | `6a40568608b7d58552d0520617c8a31a35864cb9b985fc95b6aada04cb55df6e` |
| 13+ | latest release | 1829500 | `1.4.0-arm` | 82552162 | `ba64070a3e225b8ac2ba9b1ef6a5ea3ab62c46e1c0f2f197bc3aa44b53f1495a` |
| 7–9 | nop release | 1829300 | `1.4.0-nougatOreoPie` | 77744452 | `2745dd9e18f47b78c51ae72f6bd24bee00ee73b6664d264e507b9a306fbcad7a` |
| 10–12 | qrs release | 1829400 | `1.4.0-QRS` | 84132312 | `cb66e9bd8f31c4b20c32c9f48b0353f88bc6600fc8eedd9f9b12d920d6dd0265` |

Пути: `app/build/outputs/apk/{latest/debug,latest/release,nop/release,qrs/release}/Telegram-X-Recorder-v1.4.0*.apk`. Все versionCode выше 1.3.0 (1819300–1819500) — установка поверх без удаления.

## Не проверено

Реальные звонки на устройстве (обе стороны в записи, три режима, пауза/продолжение, восстановление, воспроизведение, экспорт) и обновление поверх 1.3.0 с сохранением данных. Особенно стоит проверить: ответ на входящий из уведомления при закрытом приложении и автозапись — foreground-логика теперь upstream-овская.

Публикация (fast-forward `main`, тег `tgxr-v1.4.0`, GitHub Release) не выполнялась — ждёт решения владельца.
