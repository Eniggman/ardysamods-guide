---
name: ardysamods-guide
description: "Expert helper for Dota 2 cosmetic mods in the ArdysaMods ecosystem (Windows): Module 1 guides the player through the ArdysaModsTools launcher (install pak01_dir.vpk with Keep original, pick sets, MISCELLANEOUS map/towers/music, Patch update and quick re-patch after Dota updates); Module 2 handles emergencies and optimization: reset skins_preset.json, cut a hero or a single set out of pak01_dir.vpk with VPKEdit, or shrink a 15+ GB pack to under 1 GB with PowerShell filter scripts without breaking items_game.txt. Use when the user has ArdysaMods questions, ERROR models, crashes from a set, a broken set selector or a huge VPK. Triggers: «ArdysaMods», «моды дота 2», «как поставить арканы», «сломался выбор сетов», «вылетает дота из-за сета», «удалить сет героя», «VPKEdit», «pak01_dir.vpk», «уменьшить мод-пак», «items_game.txt»."
---

# ArdysaMods и хирургия VPK для Dota 2

Инструкция для ИИ-агента. Для людей: [README](https://github.com/Eniggman/ardysamods-guide#readme), подробные модули: [`docs/MODULE_1_USAGE_GUIDE.md`](https://github.com/Eniggman/ardysamods-guide/blob/main/docs/MODULE_1_USAGE_GUIDE.md), [`docs/MODULE_2_VPK_OPTIMIZER.md`](https://github.com/Eniggman/ardysamods-guide/blob/main/docs/MODULE_2_VPK_OPTIMIZER.md), структура VPK: [`docs/vpk_structure.md`](https://github.com/Eniggman/ardysamods-guide/blob/main/docs/vpk_structure.md).

## Маршрутизация запроса

- Установка модов и аркан, смена сета, карта, вышки, музыка, «что делать после обновления Доты», надписи ERROR → **Модуль 1** (штатная работа в лаунчере, файлы трогать не нужно).
- Сломался селектор сетов, игра вылетает из-за конкретного сета, нужно вырезать героя или сет, пак весит 15+ ГБ, вопросы про `build_filtered_tree.ps1` или `items_game.txt` → **Модуль 2**. Начинай с самого мягкого способа (0 → 1 → 2).

## Общие правила безопасности

- Перед любым изменением `pak01_dir.vpk`, `skins_preset.json` или распакованного дерева **сделай резервную копию** и получи явное «да» от человека.
- Перед правками закрыты **Dota 2 и ArdysaModsTools** (проверить в Диспетчере задач).
- После любых изменений сначала проверка в «Опробовать героя» (Demo Hero), рейтинговый матч — только потом.
- Не скачивай моды и лаунчеры из неофициальных источников. Официальные: https://github.com/Anneardysa/ArdysaModsTools, Discord (канал `#update-mods`), https://ardysamods.my.id/updates.html.

## Модуль 1. Штатный сценарий (GUI, делает человек по твоим подсказкам)

1. **Установка пака:** ArdysaModsTools → `Install mod pack` → `Manual install` → выбрать `pak01_dir.vpk` → **обязательно `Keep original`**.
2. **Сеты:** выбираются прямо в интерфейсе лаунчера, удалять файлы не нужно.
3. **Окружение:** `MISCELLANEOUS` → выбрать карту, вышки, крипов, музыку → `Generate` → **`Add to Current Mods`** (не замена!) → `Patch update` → `Verify mod file` → `Patch Update`.
4. **После микро-патча Dota 2:** `Patch update` → `Verify mod file` → `Patch Update`.
5. **После крупного патча:** взять свежий пак в Discord (`#update-mods`), установить по п. 1 и повторить п. 3.
6. Готовые пресеты в репо: `presets/MiscPreset.json`, `presets/dota.json` (окружение), `presets/skins_preset.json` (скины). Их копируют в рабочую папку лаунчера (при закрытом лаунчере), затем выполняют `Patch update` → `Verify mod file` → `Patch Update`.

FAQ: надписи **ERROR** → `Verify mod file` + `Patch Update`. Пропала карта или музыка → в `MISCELLANEOUS` было «заменить» вместо `Add to Current Mods`.

## Модуль 2. Аварийный и продвинутый сценарий

### Способ 0. Сброс `skins_preset.json` (VPK не трогаем)
1. Закрыть Dota 2 и лаунчер. Сделать копию `skins_preset.json` из рабочей папки лаунчера.
2. Найти блок проблемного героя (например, `"pudge"`) и очистить его слоты (`[]`) или заменить весь файл эталонным `presets/skins_preset.json`.
3. Лаунчер: `Patch update` → `Verify mod file` → `Patch Update`.

### Способ 1. Вырезать героя или один сет в VPKEdit GUI
1. Резервная копия: скопировать `pak01_dir.vpk` (из папки лаунчера или `…/steamapps/common/dota 2 beta/game/dota/`, бывает `game/dota_mods/`) как `pak01_dir.vpk.backup`.
2. VPKEdit (https://github.com/craftablescience/VPKEdit/releases) → `File → Open` → `pak01_dir.vpk`.
3. Удалить папки героя `<hero>` (ПКМ → Delete), только после подтверждения человека:
   `models/items/<hero>/`, `models/heroes/<hero>/`, `materials/models/items/<hero>/`, `materials/models/heroes/<hero>/`, `particles/econ/items/<hero>/`, и авторские папки, если есть: `kisilev_ind/models/<hero>/`, `8213/heroes/<hero>/`.
   Чтобы убрать **один сет**, удали только его подпапку в `models/items/<hero>/` и соответствующую в `materials/models/items/<hero>/`.
4. **Никогда не удалять** системные разделы: `scripts/` (включая `scripts/items/items_game.txt`) и другие, перечисленные в Модуле 2 (раздел 4).
5. `File → Save` (Ctrl+S), затем проверка в Demo Hero.

### Способ 2. Пакетная фильтрация по манифесту (облегчённый пак)
Требования: **PowerShell 7 (`pwsh`)**. `build_filtered_tree.ps1` использует `[System.IO.Path]::GetRelativePath`, которого нет в Windows PowerShell 5.1, хотя бейдж README обещает 5.1. Нужен также `vpkeditcli` в PATH.

1. Распаковать исходный пак в `<build_dir>\source\pak01_dir_original` через `vpkeditcli`. **Сначала выполни `vpkeditcli --help`** и сверь флаги: в Модуле 2 указаны `--extract-all`, `--save` и `--verify-checksums all`, но в официальных заметках к релизам VPKEdit встречаются `-e/--extract`, `--output` и `--verify-checksums`. Подставляй те флаги, которые реально показывает `--help` установленной версии.
2. Манифест: скопировать `presets/vpk_mod_config.template.json` в `<project_root>\vpk_mod_config.json` и вместе с человеком заполнить `active_roster`. Это **белый список**: всё, чего в нём нет, будет выброшено. Учитываются также `compat_keep_roster` и `special_assets`. Поле `deleted_heroes` скрипты не читают.
3. Построить отфильтрованную копию (исходник не меняется, папка назначения не должна существовать):
   ```powershell
   pwsh -File .\scripts\build_filtered_tree.ps1 `
     -SourceRoot "<build_dir>\source\pak01_dir_original" `
     -DestinationRoot "<build_dir>\filtered\pak01_dir" `
     -ConfigPath "<project_root>\vpk_mod_config.json"
   ```
   Скрипт выводит `Copied files: N` и `Skipped files: M`.
4. Упаковать и проверить (флаги сверь с `--help`):
   ```powershell
   vpkeditcli --save "<build_dir>\filtered\pak01_dir" --output "<project_root>\pak01_dir_rebuilt.vpk"
   vpkeditcli --verify-checksums all "<project_root>\pak01_dir_rebuilt.vpk"
   ```
5. Переименовать в `pak01_dir.vpk` и установить через лаунчер (`Install mod pack` → `Manual install` → `Keep original`).

`scripts/optimize.ps1` — **деструктивная** альтернатива: удаляет папки неактивных героев прямо в `-RootPath` (по умолчанию `.`!). Параметры: `-RootPath`, `-ConfigPath`, `-KeepKeywords`. Запускай только на копии дерева, с явно указанным `-RootPath` и после подтверждения человека. Предпочитай `build_filtered_tree.ps1`.

## Правило `items_game.txt`

- **Никогда не заменяй модовый `scripts/items/items_game.txt` дефолтным из Steam**: остальные моды сломаются.
- Не удаляй блоки и не меняй `item_def`, `event_id`, стили вручную. Типичные ошибки от таких правок: `EVENT_ID_WINTER_MAJOR_2016`, `effects_item_def 16844`.
- Сначала анализ связей (только читает файлы и пишет отчёт):
  ```bash
  python scripts/analyze_items_game_refs.py --mod <модовый items_game.txt> --default <дефолтный items_game.txt> --config vpk_mod_config.json --report items_report.md [--json-report items_report.json] [--hero lion] [--limit 200]
  ```
- `scripts/sanitize_items_game.py` в коде помечен как **Experimental only**: не сохраняет связи `event_id` и `effects_item_def`. Для рабочих сборок не используй. Если человек всё же просит, выход только в отдельный файл через `--output`, исходник не трогать.

## Проверка успеха

- Игра запускается, в Demo Hero у отредактированного героя нет ERROR, модели и эффекты на месте.
- Для Способа 2: `Skipped files` больше 0, итоговый `.vpk` заметно меньше исходного, проверка контрольных сумм проходит.

## Диагностика и откат

- Steam → Dota 2 → Свойства → Параметры запуска: `-novid -console`. В консоли искать `Failed to load model "models/items/..."`, `Cannot find material "..."`, `CPerParticleEffect: Unable to load particle system`. Путь в ошибке указывает, какой сет вырезать.
- Откат: вернуть `pak01_dir.vpk.backup`. Если бэкапа нет: Steam → Dota 2 → Свойства → Установленные файлы → «Проверить целостность файлов игры» (моды будут удалены, это решает человек).
