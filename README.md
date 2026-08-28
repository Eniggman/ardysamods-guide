# 🎮 ArdysaMods — Инструкция и инструментарий к мод-паку Dota 2

[![Dota 2](https://img.shields.io/badge/Game-Dota%202-red.svg?style=for-the-badge&logo=dota2)](https://www.dota2.com/)
[![ArdysaModsTools](https://img.shields.io/badge/Launcher-ArdysaModsTools-blue.svg?style=for-the-badge&logo=github)](https://github.com/Anneardysa/ArdysaModsTools)
[![PowerShell](https://img.shields.io/badge/Shell-PowerShell%207%20%2F%205.1-blue.svg?style=for-the-badge&logo=powershell)](https://github.com/PowerShell/PowerShell)
[![VPKEdit](https://img.shields.io/badge/CLI-VPKEdit%20CLI-orange.svg?style=for-the-badge)](https://github.com/craftablescience/VPKEdit)
[![Status](https://img.shields.io/badge/Status-Active%20%26%20Tested-brightgreen.svg?style=for-the-badge)]()

Комплексный проект документации, эталонных пресетов и скриптов автоматизации для работы с экосистемой модов **ArdysaMods** в Dota 2.

Репозиторий архитектурно разделён на **два взаимодополняющих модуля**: для обычных игроков и для продвинутых моддеров/разработчиков.

---

## 🧭 Навигация по модулям проекта

```text
ArdysaMods Toolkit
├── 📘 МОДУЛЬ 1: Базовое руководство для игроков (ArdysaModsTools)
│     └── docs/MODULE_1_USAGE_GUIDE.md  (Установка, выбор сетов в UI, ландшафты, re-patch)
│
├── ⚙️ МОДУЛЬ 2: Хирургическая оптимизация VPK (VPK Optimizer)
│     └── docs/MODULE_2_VPK_OPTIMIZER.md (Распаковка, фильтрация ассетов, сжатие с 15 ГБ до <1 ГБ)
│
├── 🗂️ СПРАВОЧНИК СТРУКТУРЫ VPK
│     └── docs/vpk_structure.md (Атлас папок, моделей, частиц, звуков и связей items_game.txt)
│
└── 🎛️ ЭТАЛОННЫЕ ПРЕСЕТЫ
      └── presets/ (MiscPreset.json, skins_preset.json, dota.json)
```

---

## 📊 Сравнение модулей: какой выбрать?

| Критерий | 📘 Модуль 1 (Игровой / UI) | ⚙️ Модуль 2 (Хирургический / VPK) |
| :--- | :--- | :--- |
| **Для кого** | Обычные игроки Dota 2 | Моддеры, энтузиасты, слабые ПК |
| **Инструмент** | Официальный лаунчер **ArdysaModsTools** | Скрипты PowerShell + **VPKEdit CLI** |
| **Выбор сетов** | Прямо в интерфейсе лаунчера | Физическое удаление файлов из архива |
| **Сложность** | 🟢 Очень просто (клик по кнопкам) | 🟡 Продвинуто (консоль, скрипты, манифесты) |
| **Размер VPK** | Полный архив (~12–18 ГБ) | Ультра-легковесный (~300–800 МБ) |
| **Ручное вырезание файлов** | ❌ **Не требуется** | ✅ Да (через `build_filtered_tree.ps1`) |
| **Ссылка на гайд** | [**Перейти к Модулю 1 ➔**](./docs/MODULE_1_USAGE_GUIDE.md) | [**Перейти к Модулю 2 ➔**](./docs/MODULE_2_VPK_OPTIMIZER.md) |

---

## 🚀 Быстрый старт

### Сценарий А: Я просто хочу скины, ландшафт и вышки в игре (Модуль 1)
1. Скачайте свежий `pak01_dir.vpk` из [Discord Ardysa Mods](https://discord.gg/GXuhAwte) (канал `#update-mods`).
2. Откройте **ArdysaModsTools** ➔ `Install mod pack` ➔ `Manual install` ➔ укажите `pak01_dir.vpk` ➔ выберите **`Keep original`**.
3. Выберите нужные сеты героям прямо в интерфейсе лаунчера.
4. Во вкладке **`MISCELLANEOUS`** выберите ландшафт, вышки, крипов, музыку ➔ `Generate` ➔ `Add to Current Mods`.
5. Перейдите в **`Patch update`** ➔ `Verify mod file` ➔ `Patch Update`. Запускайте Доту!
👉 *Полная пошаговая инструкция со скриншотами логики:* [**docs/MODULE_1_USAGE_GUIDE.md**](./docs/MODULE_1_USAGE_GUIDE.md)

---

### Сценарий Б: Мне нужно уменьшить вес мода и вырезать лишних героев (Модуль 2)
> 💡 *Внимание: в актуальном лаунчере сеты уже настраиваются в UI. Вырезать файлы вручную нужно только если вы хотите сэкономить место на SSD или создать свой легковесный пак.*

1. Распакуйте `pak01_dir_original.vpk` с помощью `vpkeditcli`.
2. Задайте список активных героев в `vpk_mod_config.json` (`active_roster`).
3. Запустите скрипт безопасной фильтрации:
   ```powershell
   scripts\build_filtered_tree.ps1 -SourceRoot .\source -DestinationRoot .\filtered -ConfigPath .\vpk_mod_config.json
   ```
4. Упакуйте `filtered` обратно в `pak01_dir.vpk` и проверьте контрольные суммы:
   ```powershell
   vpkeditcli pak01_dir.vpk --verify-checksums all
   ```
5. Установите полученный архив через ArdysaModsTools.
👉 *Полное техническое руководство и защита `items_game.txt`:* [**docs/MODULE_2_VPK_OPTIMIZER.md**](./docs/MODULE_2_VPK_OPTIMIZER.md)

---

## 📁 Структура репозитория

```text
├── docs/                               # Документация и руководства
│   ├── MODULE_1_USAGE_GUIDE.md         # Гайд для обычных игроков (ArdysaModsTools)
│   ├── MODULE_2_VPK_OPTIMIZER.md       # Руководство по хирургической оптимизации VPK
│   └── vpk_structure.md                # Справочник внутренней структуры папок VPK
├── presets/                            # Эталонные файлы конфигурации
│   ├── MiscPreset.json                 # Пресет карты, вышек, крипов и музыки (MISCELLANEOUS)
│   ├── skins_preset.json               # Пресет выбранных сетов и Аркан на героев
│   ├── dota.json                       # Конфигурация лаунчера
│   ├── vpk_mod_config.template.json    # Шаблон манифеста для сборки легковесного VPK
│   └── README.md                       # Описание пресетов и инструкция по установке
├── scripts/                            # Скрипты автоматизации
│   ├── build_filtered_tree.ps1         # Основной скрипт сборки чистого дерева по манифесту
│   ├── optimize.ps1                    # Скрипт in-place зачистки каталогов
│   ├── analyze_items_game_refs.py      # Анализатор связей и целостности items_game.txt
│   └── sanitize_items_game.py          # Экспериментальный санитайзер schema
├── README.md                           # Главная страница проекта
└── SKILL.md                            # Описание навыка для AI-ассистента Antigravity
```

---

## 🌐 Официальные ресурсы
- [ArdysaModsTools GitHub](https://github.com/Anneardysa/ArdysaModsTools) — официальный лаунчер модов.
- [Discord (Ardysa Mods)](https://discord.gg/GXuhAwte) — официальное сообщество и свежие VPK (канал #update-mods).
- [ArdysaMods Updates](https://ardysamods.my.id/updates.html) — витрина обновлений и предпросмотр сетов.
- [VPKEdit Releases](https://github.com/craftablescience/VPKEdit/releases) — CLI и GUI инструмент для работы с VPK архивами Valve.
