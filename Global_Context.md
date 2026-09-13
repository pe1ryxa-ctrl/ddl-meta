# Danger Drones Lab (DDL) — Global Context

## Екосистема DDL

Екосистема Danger Drones Lab складається з п'яти незалежних, але тісно інтегрованих проєктів, які працюють разом для забезпечення керування, налаштування та моніторингу FPV-дронів:

1.  **DDL Server (Backend & Portal)**
    *   **Роль:** Центральний вузол зв'язку, авторизації (SSO), збереження прошивок (OTA) та веб-портал.
    *   **Технології:** FastAPI (Python), PostgreSQL, Redis, Vanilla JS SPA, Authentik, Cloudflare R2.
    *   **Взаємодія:** Надає REST API та WebSerial Web Flasher для хардверних модулів (STC, SRC), Web UI для користувачів, дані для DAI.

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
    *   **Технології:** Python 3.12 / FastAPI (монолітний `dai_backend.py`, macOS); інференс — локальна `Gemma 12B` через Ollama Tool Calling (Agentic Multi-hop RAG, `OLLAMA_NUM_CTX` 32K); база знань — Pure OKF Markdown-wiki (`data/fpv_wiki/`) з лексичним пошуком `bm25s` (вектори/ембеддінги відкинуто); історія чату — SQLite (`chat_history.db`, SQL LIKE); генерація wiki та Vision — Gemini API (`gemini-3.1-flash-lite` масово, `gemini-3.1-pro-preview` для repair-агента); харвестинг Telegram — Telethon (MTProto); ProcessGuard (PID-локи, ліміти API).
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

- **SSOT:** Holy Trinity (`Context_X / Plan_X / Changelog_X`) веде L1; `Global_Context.md`, `Global_Roadmap.md`, `Global_Changelog.md` веде Claude. Plan містить лише беклог.
- **Задачі:** шаблон `DDL/.agents/TASK_TEMPLATE.md`; ID — `<TAG>-<NNN>` (DDL, STC, SRC, DAI, HUB); файли комітяться в git проєкту; закриті — у `tasks/done/`.
- **OKF Sync Protocol:** якщо в SRC/STC оновлено `USER_GUIDE.md` або вирішено апаратну проблему (L1 зазначає це в Report), Claude ставить задачу L1 DAI на перенесення знань у `DAI/data/fpv_wiki/` (формат OKF) та поповнення `hardware_troubleshooting.md`.

## Квота L1
Облік витрат тижневої квоти Antigravity (Gemini / Claude+GPT) — `L1_Quota_Ledger.md`; скриншот «Models & Usage» після кожного Report. У тиждень дефіциту механічні задачі бере Архітектор (виняток із «не пише код», diff перевіряють незалежні рев'ю-агенти, приймає Gans).
