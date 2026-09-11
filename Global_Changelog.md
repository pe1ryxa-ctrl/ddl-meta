# Danger Drones Lab (DDL) — Global Changelog

Цей файл містить історію мажорних (крос-проєктних) релізів екосистеми DDL. Детальні зміни кожного окремого компонента знаходяться у відповідних локальних `Changelog_*.md` (Server, STC, SRC, DAI).

## [Unreleased]

## [v2.1.0] - 2026-09-10
### Process
* **Ecosystem:** Перехід AI-воркфлоу на схему Claude = L2 Architect / Gemini (Antigravity) = L1 Developer. Dual Persona Mode (`/l1_mode`, `l2-architect-enforcer`, підписи за режимом) скасовано.
* **Process:** Обмін ТЗ/звітами через файли-задачі `<P>/.agents/tasks/<ID>.md` (шаблон `DDL/.agents/TASK_TEMPLATE.md`) та воркфлоу Antigravity `/task <ID>` замість копіпасту через керівника. Верифікація звітів L1 — Архітектором по `git diff`.
* **Docs:** Новий `DDL/CLAUDE.md` (протокол Архітектора). `<P>/.agents/AGENTS.md` переписано як контракт L1; `project-rules.md` — лише технічні правила; `~/.gemini/config/AGENTS.md` — лише L1. З 15 скілів вирізано секції DUAL PERSONA; `vtx-skill-updater` — тепер обов'язок Claude.
* **Cleanup:** `Server/.agents/rules/architect_prompt.md`, `SRC/.cursorrules`, teamwork-залишки `DDL/.agents/` та скіл `l2-architect-enforcer` перенесено в `DDL/_archive/`.

## [v2.0.2] - 2026-08-22
### Security
* **DAI:** SSRF redirect block and URL normalizer security fixes.
* **DAI:** Blocked ArduPilot ChibiOS `.abin`, `.rom`, and `.bootloader` binaries from being scraped and cleaned up existing garbage.
### Added
* **DAI:** Native Office Document parsing (docx, xlsx).
* **DAI:** Git Parser overhaul with hwdef.dat support.
### Fixed
* **DAI:** Blocked .elf/.apj binaries from processing.
* **DAI:** Fixed URL normalizer query stripping.

## [v2.0.1] - 2026-08-22
### Додано / Змінено
* **DAI:** Повна стабілізація RAG-пайплайну (Async Parallel Processing, атомарні транзакції, захист від втрати даних).
* **DAI:** Успішна міграція всієї архітектури на оптимізовану модель `gemini-3.1-flash-lite` з каскадним фолбеком.
* **DAI:** Впроваджено інтелектуальну фільтрацію сміттєвого контенту (форумні бейджики, профілі, специфічні бінарники дронів) для збереження квот API.
* **DAI:** Усунено спам-повідомлення в терміналі через коректне архітектурне вимкнення Automatic Function Calling у `google.genai` SDK та міграцію на сучасний API `pymupdf`.

## [v2.0.0] - 2026-08-21
### Додано / Змінено
* **Ecosystem:** Перехід на 4-компонентну архітектуру (Server, STC, SRC, DAI).
* **Process:** Впроваджено єдину систему ведення `Plan.md` (чистий беклог без дублювання виконаних задач) та глобальні SSOT (Global_Context.md, Global_Roadmap.md, Global_Changelog.md).
* **Server:** Інтеграція безпечного чату на базі WebSockets замість Matrix Conduit.
* **Infrastructure:** Глобальна міграція всієї кодової бази на GitHub.
* **SRC:** Стабілізація прошивки `v2.0.0-stable` (тестування).
* **DAI:** Міграція на macOS, інтеграція Gemma 12B з динамічним контекстом 128K.
