# Danger Drones Lab (DDL) — Global Context

## Екосистема DDL

Екосистема Danger Drones Lab складається з п'яти незалежних, але тісно інтегрованих проєктів, які працюють разом для забезпечення керування, налаштування та моніторингу FPV-дронів:

1.  **DDL Server (Backend & Portal)**
    *   **Роль:** Центральний вузол зв'язку, авторизації (SSO), збереження прошивок (OTA) та веб-портал.
    *   **Технології:** FastAPI (Python), PostgreSQL, Redis, Vanilla JS SPA, Authentik, Cloudflare R2.
    *   **Взаємодія:** Надає REST API та WebSerial Web Flasher для хардверних модулів (STC, SRC), Web UI для користувачів, дані для DAI.
    *   **Деплой (з 26.09.2026, DDL-004):** пуш коду в `main` → тести на Python 3.12 → деплой рівно перевіреного коміту на VPS (`alembic upgrade head` включно). Пуш лише `.agents/`, кореневих `*.md`, `brain/` деплою не запускає. Інших способів деплою немає.

2.  **STC (Smart Transmitter Control / VTX)**
    *   **Роль:** Бортовий модуль керування відеопередавачем (VTX) на дроні (ESP32-C3). Підтримує протоколи SmartAudio та Tramp.
    *   **Технології:** ESP-IDF, FreeRTOS, One-Wire UART (SmartAudio v2.1, Tramp), OLED SSD1306, Secure Boot V2.
    *   **Взаємодія:** Отримує команди зміни частоти/потужності від RC-приймача (через CRSF/SBUS канали польотного контролера). Оновлюється через OTA з DDL Server.

3.  **SRC (Smart Remote Control / VRX)**
    *   **Роль:** Наземний модуль на стороні пілота для керування відеоприймачем (VRX) Skyzone SteadyView (ESP32-C3). Забезпечує узгодження частоти з VTX через RC-пресети.
    *   **Технології:** ESP-IDF, FreeRTOS, CRSF/SBUS/S.Port/F.Port парсинг, OLED SSD1306.
    *   **Взаємодія:** Приймає RC-дані від пульта пілота (через CRSF UART), перемикає канали VRX відповідно до RC-пресетів. Оновлюється через OTA з DDL Server. Зв'язок між SRC та STC здійснюється опосередковано — через RC-канали польотного контролера та пульта, а не прямим радіолінком.

4.  **DAI (Danger AI)**
    *   **Роль:** Автономний AI-бот-асистент для FPV-пілотів: підключається WebSocket-клієнтом до DDL Server (`/dai` firehose), відповідає в чаті спільноти, збирає та синтезує базу знань з веб-джерел, форумів і Telegram-чатів.
    *   **Технології:** Python 3.14 (Homebrew 3.14.7; лінива оцінка анотацій ховає помилки, що на старших версіях падають на імпорті — перевіряти перед міграцією на інший ПК) / FastAPI (монолітний `dai_backend.py`, macOS); інференс — локальна `Gemma 12B` через Ollama Tool Calling (Agentic Multi-hop RAG, `OLLAMA_NUM_CTX` 32K); база знань — Pure OKF Markdown-wiki (`data/fpv_wiki/`) з лексичним пошуком `bm25s` (вектори/ембеддінги відкинуто); історія чату — SQLite (`chat_history.db`, SQL LIKE); генерація wiki та Vision — Gemini API (`gemini-3.1-flash-lite` масово, `gemini-3.1-pro-preview` для repair-агента); харвестинг Telegram — Telethon (MTProto); ProcessGuard (PID-локи, ліміти API).
    *   **Взаємодія:** Отримує тригери й надсилає відповіді через WebSocket DDL Server (усі фрейми з `branch_id`); синхронізує знання з STC/SRC (`USER_GUIDE.md`, апаратні кейси) через OKF Sync Protocol.

5.  **Sensor HUB (Sensor Fusion Hub)**
    *   **Роль:** Автономний модуль сенсорної фузії (RP2350, Pico 2 W) для ArduPilot: EKF, MAVLink DMA, Wi-Fi-сканер, OTA. Розробка під HIL-гейтом.
    *   **Технології:** Pico SDK 2.x, FreeRTOS SMP, C++17, CMake, RP2350 Secure/Encrypted Boot.
    *   **Взаємодія:** MAVLink з польотним контролером; знання й правила — `Sensor HUB/.agents/okf/`.

## Ролі та AI-воркфлоу (з 2026-09-10)

| Хто | Роль |
|---|---|
| **Gans** | Керівник. Затверджує ТЗ, тригерить L1 (`/task <ID>`), HIL-тестує залізо, дає згоду на ADLP-дії та деплої. |
| **Claude** | **L2 Architect.** Планує, пише задачі у `<P>/.agents/tasks/<ID>.md`, верифікує роботу L1 по `git diff`, веде Global_* та всі `.agents/`-конфіги і скіли Gemini. Не пише код проєктів. Протокол: `DDL/CLAUDE.md`. |
| **Gemini (Antigravity)** | **L1 Developer.** Один воркспейс = один проєкт. Реалізує, тестує, оновлює Holy Trinity, комітить, дописує Report у task-файл. Контракт: `<P>/.agents/AGENTS.md`, глобально — `~/.gemini/config/AGENTS.md`. |

Dual Persona Mode (одна модель у двох ролях через `/l1_mode`) скасовано: роль визначається інструментом. Стара конфігурація — `DDL/_archive/`.

- **Підлеглі сесії Claude (з 2026-09-29, протокол kit `28f1c09`).** Основна сесія Архітектора — на Mac («Виправлення DAI»): проєкти, бойовий бот і його `.env`, злиття, задачі, виконавці. Підлегла сесія «ПК-90» (Windows-ПК з RTX 5060 8 ГБ, процесор AMD Ryzen 7 8845HS — 8 ядер / 16 потоків, Zen 4, вбудована Radeon 780M; ОЗП 32 ГБ; SSD 1 ТБ; з даних Gans 03.10; Remote Control) тримає лише сам ПК-90: Ollama з `gemma4:26b` (модель бота по LAN), завдання Планувальника від SYSTEM без входу користувача (з 29.09, T-OLLAMA-BOOT: `ollama serve` і прогрів 26B при старті Windows, конфіг `C:\Antigravity\ollama-boot`, моделі `D:\Ollama`, контекст 32768; вимкнення о 21:50 (з 01.10; `shutdown /t 120` → ~21:52); журнал `/api/ps` о 08:35 і 12:00; BIOS-увімкнення о 08:20 (з 02.10); активні години Windows Update 08:00–22:00; перевірено перезавантаженням 21:17 — модель за 50 с), локальні заміри й прогони тестів на Windows. Вона не комітить в основні гілки. Будь-яку зміну моделі на ПК-90 спершу погоджує Gans (бот користується нею наживо). Поділ погодив Gans 29.09.
- **DAI на Windows не підтримується (рішення Gans 01.10, T-WIN закрито).** Ціль DAI — macOS; ТЗ DAI вимагають macOS і Linux (Linux покривають хмарні виконавці). На ПК-90 Smart App Control (On) блокує непідписаний `uuid_utils` (.pyd) → langsmith → langchain_core → `scripts/wiki_agent.py`: код DAI на Windows не імпортується; SAC не вимикаємо (може бути незворотно). Знахідки T-WIN-1/2 (W1 `os.geteuid` у skipif `test_dai_038_run_ledger.py:255`, W2 `import resource` у `kb_quality_audit.py:46`, W3–W5 bash-тести під Git Bash, W6 SAC) — довідково, задач не ставимо. Сесія ПК-90 стає досяжною (Remote Control) лише після відкриття сесії Gans; до того повідомлення чекають у черзі.
- **SSOT:** Holy Trinity (`Context_X / Plan_X / Changelog_X`) веде L1; `Global_Context.md`, `Global_Roadmap.md`, `Global_Changelog.md` веде Claude. Plan містить лише беклог.
- **Задачі:** шаблон `DDL/.agents/TASK_TEMPLATE.md`; ID — `<TAG>-<NNN>` (DDL, STC, SRC, DAI, HUB); файли комітяться в git проєкту; закриті — у `tasks/done/`.
- **OKF Sync Protocol:** якщо в SRC/STC оновлено `USER_GUIDE.md` або вирішено апаратну проблему (L1 зазначає це в Report), Claude ставить задачу L1 DAI на перенесення знань у `DAI/data/fpv_wiki/` (формат OKF) та поповнення `hardware_troubleshooting.md`.

## Квота L1
Облік витрат тижневої квоти Antigravity (Gemini / Claude+GPT) — `L1_Quota_Ledger.md`; скриншот «Models & Usage» після кожного Report. **L2-фолбек — лише за явним розпорядженням Gans на конкретну задачу** (протокол `DDL/CLAUDE.md` §4): виконує субагент Архітектора, коміти `[L2 fallback]`, рев'ю — інший субагент. Архітектор фолбек як спосіб розвантажити чергу не пропонує; дефіцит квоти — привід для черги або паузи, не для фолбеку.
