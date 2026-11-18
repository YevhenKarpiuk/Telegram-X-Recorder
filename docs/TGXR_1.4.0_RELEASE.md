# Telegram X Recorder 1.4.0

Recorder обновлён с Telegram X 0.29.1.1819 до **0.29.2.1829-beta**.

- Upstream: `7fe047c8861677845621922522c651bac6924d0c` (тег `v0.29.2.1829-beta`).
- Код APK: `e7dbd541`, ветка `main`.
- Package: `ka.soft.tgxr`, подпись совпадает с Recorder 1.2.0 и 1.3.0. Ставится поверх, удалять приложение не нужно.
- Каждый APK содержит ARM32 и ARM64.

## Что нового

- Все изменения Telegram X 1829: переработанные настройки звонков и раздел проблем со звонками, запрос микрофона при ответе, исправление входящих при закрытом приложении, защита входящих звонков паролем, звонки избранных контактов и повторные звонки в режиме «Не беспокоить», новый WebRTC с исправлением падения при невозможности начать запись звука, TDLib `c8828d5`, tlottie вместо rlottie.
- tgcalls обновлён до TGX production `97c1537` (TelegramMessenger/tgcalls@ea5153e); хуки записи сохранены без изменений.
- **Bluetooth (Android 12+):** звонок сразу идёт в подключённую гарнитуру, во время набора устройство больше не переключается само туда-сюда, после звонка записи снова проигрываются через Bluetooth. Это исправление ошибки самого Telegram X.
- Настройки записи (автозапись, файлы записи, список записей) перенесены в **Настройки → Уведомления → Звонки**.
- Формат и каталог записей, режимы, пауза/продолжение, восстановление, воспроизведение, экспорт и удаление не менялись.

## Скачать

[GitHub Release tgxr-v1.4.0](https://github.com/YevhenKarpiuk/Telegram-X-Recorder/releases/tag/tgxr-v1.4.0)

| Android | APK | versionCode |
| --- | --- | ---: |
| 13+ | [Telegram-X-Recorder-v1.4.0.apk](https://github.com/YevhenKarpiuk/Telegram-X-Recorder/releases/download/tgxr-v1.4.0/Telegram-X-Recorder-v1.4.0.apk) | 1829500 |
| 10–12 | [Telegram-X-Recorder-v1.4.0-QRS.apk](https://github.com/YevhenKarpiuk/Telegram-X-Recorder/releases/download/tgxr-v1.4.0/Telegram-X-Recorder-v1.4.0-QRS.apk) | 1829400 |
| 7–9 | [Telegram-X-Recorder-v1.4.0-nougatOreoPie.apk](https://github.com/YevhenKarpiuk/Telegram-X-Recorder/releases/download/tgxr-v1.4.0/Telegram-X-Recorder-v1.4.0-nougatOreoPie.apk) | 1829300 |

Рядом с каждым APK опубликован файл `.sha256`, общий список — `SHA256SUMS.txt`.

## Сборка

Новое требование upstream: Rust 1.99.1 с Android-таргетами (`scripts/setup-rust.sh`, `rust.dir` в `local.properties`). Flavor для Android 7–9 называется теперь `nop` (было `tens`).

## Проверки

Debug и три Release собраны, 27 unit tests прошли. Проверены package, versionCode, обе ABI, JNI-экспорты записи, DEX, выравнивание 16 KiB и сертификат. На телефоне владельца проверены звонки и запись через Bluetooth, маршрутизация гарнитуры и проигрывание записей после звонка.

[Подробный отчёт интеграции](TGX_1829_RECORDER_UPGRADE.md).
