# ⚙️ Модуль 2: Хирургическая оптимизация VPK (VPK Optimizer & Asset Purge)

> **⚠️ ВАЖНЫЙ ДИСКЛЕЙМЕР:** В актуальных версиях ArdysaModsTools выбор и отключение скинов доступны прямо в графическом интерфейсе (см. [Модуль 1](./MODULE_1_USAGE_GUIDE.md)).
>
> Этот модуль — **продвинутый инструмент для разработчиков, моддеров и энтузиастов**, которым необходимо:
> 1. **Физически уменьшить вес архива VPK** с 12–18 ГБ до менее чем 1 ГБ (экономия места на SSD, мгновенная загрузка игры).
> 2. **Полностью удалить из VPK файлы неиспользуемых героев** (модели, текстуры, эффекты, звуки), оставив только пул мейн-героев.
> 3. **Глубоко кастомизировать ассеты** через скрипты PowerShell и VPKEdit CLI с гарантией целостности `items_game.txt`.

---

## 📋 Оглавление
1. [Архитектура VPK и структура ассетов Dota 2](#1-архитектура-vpk-и-структура-ассетов-dota-2)
2. [Критическое правило items_game.txt и Valve Econ Schema](#2-критическое-правило-items_gametxt-и-valve-econ-schema)
3. [Уроки и проверенные ошибки (Lessons Learned)](#3-уроки-и-проверенные-ошибки-lessons-learned)
4. [Конфигурационный манифест (vpk_mod_config.json)](#4-конфигурационный-манифест-vpk_mod_configjson)
5. [Пошаговый процесс оптимизации (Workflow)](#5-пошаговый-процесс-оптимизации-workflow)
6. [Инструментарий и скрипты репозитория](#6-инструментарий-и-скрипты-репозитория)
7. [Интеграция с ArdysaModsTools](#7-интеграция-с-ardysamodstools)

---

## 1. Архитектура VPK и структура ассетов Dota 2

В Dota 2 косметика распределена по множеству виртуальных каталогов. Для полного удаления ассетов конкретного героя необходимо очистить его файлы во всех следующих разделах:

### 👤 Пути ассетов героев:
- **Базовые модели:** `models/heroes/<hero_name>/`
- **Арканы и сеты предметов:** `models/items/<hero_name>/` (например, Аркана Earthshaker, сеты Anti-Mage и т.д.)
- **Текстуры и материалы:** `materials/models/heroes/<hero_name>/`, `materials/models/items/<hero_name>/`
- **Частицы и эффекты способностей:** `particles/units/heroes/hero_<hero_name>/`, `particles/econ/items/<hero_name>/`
- **Звуки и озвучка:** `sounds/weapons/hero/<hero_name>/`, `sounds/vo/<hero_name>/`
- **Кастомные пространства имён авторов мода:** `kisilev_ind/models/<hero_name>/`, `8213/heroes/<hero_name>/`, `jxj/models/heroes/<hero_name>/`

### 🛡️ Системные разделы (СТРОГО СОХРАНЯТЬ):
Ни при каких условиях **НЕЛЬЗЯ фильтровать или удалять** следующие папки из корня архива:
- `panorama/` — интерфейс, меню, HUD, экраны выбора.
- `scripts/` — игровая логика, манифесты и `scripts/items/items_game.txt`.
- `maps/` — геометрия карты и ландшафты.
- `music/`, `soundevents/` — звуковые моды и музыкальные события.
- `models/props_structures/` — кастомные башни (Towers).
- `materials/environment/` — текстуры ландшафта и окружения.

---

## 2. Критическое правило items_game.txt и Valve Econ Schema

Файл `scripts/items/items_game.txt` — это не просто список путей к моделям, а **глобальная реляционная база данных Valve Econ Schema**.

### Почему это критично:
- Каждый косметический предмет имеет уникальный идентификатор (`item_def`).
- Внутри схемы существуют перекрёстные связи между предметами, событиями (`event_id`), эффектами (`effects_item_def`), бандлами (`bundles`), стилями (`styles`) и модификаторами ассетов (`asset_modifier`).
- Если вы оставите модовый `items_game.txt`, но физически удалите 3D-модели и текстуры неактивных героев, Valve Schema попытается загрузить несуществующие модовые файлы, что вызовет появление красных надписей **ERROR** или невидимость обычных скинов у неактивных героев.
- Для strict lightweight-сборки это **не решается** возвратом файлов неактивных героев обратно в архив. Если героя нет в активном списке (`active_roster`), его файлов не должно быть в VPK.

---

## 3. Уроки и проверенные ошибки (Lessons Learned)

В ходе разработки и тестирования хирургического оптимизатора были детально исследованы различные подходы:

| Подход | Результат | Статус |
| :--- | :--- | :--- |
| **Возврат ассетов через compat_keep_roster** | Устраняет баги отображения, но раздувает VPK до 10+ ГБ, нарушая цель lightweight-сборки. | ❌ Отклонено |
| **Массовая замена item-блоков на дефолтные** | Fatal Crash при запуске Dota 2: `EVENT_ID_WINTER_MAJOR_2016` / `Unable to find effects_item_def 16844`. | ❌ Запрещено |
| **Точечная замена одиночного блока (Lion 572)** | Игра запускается, но остаются модовые иконки/эффекты и отсутствующие модели. | ❌ Неэффективно |
| **Сборка через Copy-Filter (новое чистое дерево)** | Полная стабильность VPK, корректные ACL права, защита исходного архива от повреждений. | ✅ **Эталонный метод** |

### 🛑 Золотые правила работы с items_game.txt:
1. **Никогда не заменяйте item-блоки целиком** на дефолтные блоки из установленной Dota 2.
2. Не модифицируйте `event_id`, item id, структуру бандлов, стилей и econ-полей.
3. Если требуется адаптация `items_game.txt` для выключенных героев — точечно заменяйте только конкретные строковые пути (`image_inventory`, `model_player`, `particle`, `asset_modifier`).

---

## 4. Конфигурационный манифест (vpk_mod_config.json)

Манифест `vpk_mod_config.json` является **единственным источником правды** для скриптов фильтрации.

### Пример конфигурации:
```json
{
  "project": "Dota 2 Lightweight VPK",
  "version": "1.1.0",
  "last_updated": "2026-08-28",
  "active_roster": [
    "wisp",
    "io",
    "sniper",
    "razor",
    "muerta",
    "phantom_lancer",
    "luna",
    "earthshaker",
    "legion_commander"
  ],
  "compat_keep_roster": [],
  "special_assets": [
    "portal",
    "cube",
    "companion",
    "companion_cube"
  ],
  "deleted_heroes": [
    "abaddon",
    "abyssal_underlord",
    "alchemist",
    "..."
  ]
}
```

### Алиасы героев и спец-активы:
Скрипт `build_filtered_tree.ps1` автоматически нормализует и учитывает альтернативные имена персонажей:
- **Io / Wisp:** `wisp`, `io`, `wips`, а также ассеты Арканы: `portal`, `cube`, `companion`, `companion_cube`.
- **Sniper:** `sniper`, `kardel`.
- **Phantom Lancer:** `phantom_lancer`, `phantomlancer`.
- **Bounty Hunter:** `bounty_hunter`, `bountyhunter`, `gondar`.
- **Защита от ложных срабатываний:** сравнение токенов предотвращает ошибочное удаление героев с похожими именами (`io` не удаляет `lion`, `furion`, `legion_commander`).

---

## 5. Пошаговый процесс оптимизации (Workflow)

```text
[ pak01_dir_original.vpk ]
            |
            v  (1. Распаковка через vpkeditcli)
[ build/source/pak01_dir_original/ ]
            |
            v  (2. Фильтрация через build_filtered_tree.ps1 + vpk_mod_config.json)
[ build/filtered/pak01_dir/ ]
            |
            v  (3. Упаковка через vpkeditcli)
[ pak01_dir_rebuilt.vpk ]
            |
            v  (4. Верификация vpkeditcli --verify-checksums)
[ Установка в ArdysaModsTools ]
```

### Шаг 1: Распаковка оригинального архива
Используйте VPKEdit CLI для полной распаковки эталонного VPK:
```powershell
vpkeditcli --extract-all "D:\Coding\VPK_Project\pak01_dir_original.vpk" --output "D:\Coding\VPK_Project\build\source\pak01_dir_original"
```

### Шаг 2: Настройка списка активных героев
Отредактируйте `vpk_mod_config.json`, указав в массиве `active_roster` только тех героев, скины которых вам действительно нужны.

### Шаг 3: Построение фильтрованного дерева
Запустите скрипт сборки чистого дерева:
```powershell
.\scripts\build_filtered_tree.ps1 `
  -SourceRoot "D:\Coding\VPK_Project\build\source\pak01_dir_original" `
  -DestinationRoot "D:\Coding\VPK_Project\build\filtered\pak01_dir" `
  -ConfigPath "D:\Coding\VPK_Project\vpk_mod_config.json"
```

### Шаг 4: Упаковка нового VPK
Соберите оптимизированное дерево обратно в архив:
```powershell
vpkeditcli --save "D:\Coding\VPK_Project\build\filtered\pak01_dir" --output "D:\Coding\VPK_Project\pak01_dir_rebuilt.vpk"
```

### Шаг 5: Проверка целостности
Проверьте контрольные суммы полученного архива:
```powershell
vpkeditcli --verify-checksums all "D:\Coding\VPK_Project\pak01_dir_rebuilt.vpk"
```

### Шаг 6: Замена рабочего файла
Сделайте резервную копию старого рабочего VPK и замените его на свежесобранный `pak01_dir_rebuilt.vpk`.

---

## 6. Инструментарий и скрипты репозитория

В папке [`scripts/`](../scripts/) содержатся специализированные утилиты:

- 🚀 **[`scripts/build_filtered_tree.ps1`](../scripts/build_filtered_tree.ps1)** — **Основной рабочий инструмент.** Безопасно копирует только разрешённые файлы из распакованного оригинала в чистое дерево по манифесту.
- 🧹 **[`scripts/optimize.ps1`](../scripts/optimize.ps1)** — Вспомогательный скрипт для in-place очистки уже существующего каталога (удаляет папки неактивных героев на месте).
- 🔍 **[`scripts/analyze_items_game_refs.py`](../scripts/analyze_items_game_refs.py)** — Python-анализатор ссылочной целостности `items_game.txt` (проверяет перекрёстные ссылки между item-блоками, событиями и модификаторами).
- 🧪 **[`scripts/sanitize_items_game.py`](../scripts/sanitize_items_game.py)** — Экспериментальный прототип санитайзера (оставлен для исследовательских целей).

---

## 7. Интеграция с ArdysaModsTools

После сборки оптимизированного `pak01_dir.vpk`:
1. Откройте **ArdysaModsTools**.
2. Выполните установку через **Install mod pack** -> **Manual install** -> **Keep original** (подробно описано в [Модуле 1](./MODULE_1_USAGE_GUIDE.md#2-шаг-1-правильная-установка-мод-пака-install-mod-pack)).
3. Сгенерируйте окружение через **MISCELLANEOUS** -> **Generate** -> **Add to Current Mods**.
4. Завершите процесс через **Patch update** -> **Verify mod file** -> **Patch Update**.

---

## 🔗 Связанные материалы
- [Модуль 1: Базовое руководство для игроков](./MODULE_1_USAGE_GUIDE.md)
- [Справочник структуры папок VPK](./vpk_structure.md)
- [Шаблон манифеста vpk_mod_config](../presets/vpk_mod_config.template.json)
- [Каталог пресетов](../presets/README.md)