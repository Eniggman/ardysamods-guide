name: dota-surgical-vpk-optimizer
description: Комплексный инструментарий и экспертный ассистент по экосистеме ArdysaMods для Dota 2. Включает Модуль 1 (базовое руководство для игроков по ArdysaModsTools, выбору сетов в UI, настройке окружения MISCELLANEOUS и быстрому обновлению патча) и Модуль 2 (глубокая хирургическая оптимизация и пересборка VPK через PowerShell, фильтрация ассетов по манифесту, защита ссылочной целостности items_game.txt и econ schema).
---

# ArdysaMods & Dota 2 Surgical VPK Toolkit

Этот навык оснащает ИИ-ассистента полной экспертизой по работе с модами Dota 2 в экосистеме **ArdysaMods**.

Навык поддерживает два основных сценария взаимодействия:
1. **МОДУЛЬ 1 (Базовый сценарий для игроков):** Пошаговая консультация и помощь в работе с графическим лаунчером **ArdysaModsTools** (установка `pak01_dir.vpk`, выбор сетов в UI, настройка ландшафта, вышек и музыки во вкладке `MISCELLANEOUS`, применение `Patch update`, быстрый re-patch при обновлениях Dota 2).
2. **МОДУЛЬ 2 (Продвинутый сценарий для разработчиков и оптимизаторов):** Хирургическая пересборка архивов VPK для кардинального снижения веса мода (с 15+ ГБ до <1 ГБ), фильтрация файлов через `scripts/build_filtered_tree.ps1` по манифесту `vpk_mod_config.json`, упаковка через `vpkeditcli` и предотвращение повреждений `scripts/items/items_game.txt`.

---

## 🧭 Маршрутизация запросов пользователя

### 1. Если пользователь спрашивает:
- «Как установить моды / Арканы?»
- «Как настроить карту, вышки или музыку?»
- «Как поменять скин на герое в лаунчере?»
- «Что делать после обновления Доты?»
- «Почему в игре надписи ERROR?»
👉 **Используйте инструкции МОДУЛЯ 1** ([`docs/MODULE_1_USAGE_GUIDE.md`](./docs/MODULE_1_USAGE_GUIDE.md)).
- Рекомендуйте использовать **ArdysaModsTools**.
- Напоминайте при установке выбирать `Keep original`.
- Объясняйте, что сеты переключаются прямо в интерфейсе без необходимости удалять файлы.
- Для окружения напоминайте последовательность: `MISCELLANEOUS` -> `Generate` -> `Add to Current Mods` -> `Patch update` -> `Verify mod file` -> `Patch Update`.

### 2. Если пользователь спрашивает:
- «Как вырезать лишних героев из VPK?»
- «Как уменьшить размер пака до 1 ГБ?»
- «Как пересобрать VPK через скрипт?»
- «Как работает build_filtered_tree.ps1?»
- «Почему падает игра при изменении items_game.txt?»
👉 **Используйте инструкции МОДУЛЯ 2** ([`docs/MODULE_2_VPK_OPTIMIZER.md`](./docs/MODULE_2_VPK_OPTIMIZER.md)).
- Напоминайте дисклеймер: в лаунчере есть встроенный переключатель, но для физического удаления файлов используется сборка через чистое filtered-дерево.
- Используйте скрипт `scripts/build_filtered_tree.ps1` и манифест `vpk_mod_config.json`.
- Предупреждайте о недопустимости массовой замены item-блоков в `items_game.txt` (ошибка `EVENT_ID_WINTER_MAJOR_2016` / `Unable to find effects_item_def 16844`).

---

## 📁 Структура и ключевые ресурсы репозитория

- 📘 **[`docs/MODULE_1_USAGE_GUIDE.md`](./docs/MODULE_1_USAGE_GUIDE.md)** — Подробный гайд для игроков по ArdysaModsTools.
- ⚙️ **[`docs/MODULE_2_VPK_OPTIMIZER.md`](./docs/MODULE_2_VPK_OPTIMIZER.md)** — Продвинутый гайд по оптимизации и сборке VPK.
- 🗂️ **[`docs/vpk_structure.md`](./docs/vpk_structure.md)** — Справочник структуры файлов и виртуальных путей Dota 2.
- 🎛️ **[`presets/`](./presets/)**:
  - `MiscPreset.json` — конфигурация карты, вышек, крипов и музыки.
  - `skins_preset.json` — конфигурация выбранных скинов персонажей.
  - `dota.json` — конфигурация лаунчера.
  - `vpk_mod_config.template.json` — шаблон манифеста для фильтрации VPK.
- 🛠️ **[`scripts/`](./scripts/)**:
  - `build_filtered_tree.ps1` — сборка чистого filtered-дерева из распакованного архива.
  - `optimize.ps1` — in-place очистка каталогов.
  - `analyze_items_game_refs.py` — анализ целостности связей `items_game.txt`.
  - `sanitize_items_game.py` — экспериментальный санитайзер.

---

## ⚡ Быстрые команды для ИИ-ассистента

### Построение фильтрованного дерева:
```powershell
.\scripts\build_filtered_tree.ps1 `
  -SourceRoot "<build_dir>\source\pak01_dir_original" `
  -DestinationRoot "<build_dir>\filtered\pak01_dir" `
  -ConfigPath "<project_root>\vpk_mod_config.json"
```

### Упаковка и верификация через VPKEdit CLI:
```powershell
vpkeditcli --save "<build_dir>\filtered\pak01_dir" --output "<project_root>\pak01_dir_rebuilt.vpk"
vpkeditcli --verify-checksums all "<project_root>\pak01_dir_rebuilt.vpk"
```
