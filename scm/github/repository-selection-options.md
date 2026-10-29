# Options SCM-отбора (GitHub)

Extension-флаги SCM-отбора, имеющие смысл **только** при `upstream=github` после
резолва: контракт CLI, ключи профиля и **продуктовый эффект** без REST.

Переносимые критерии и `scope` — [hub SCM-отбора][scm-repository-selection].
Маппинг параметров на HTTP — [repository-selection-mapping/index.md](repository-selection-mapping/index.md).
Раскладка слоёв — [ADR-26102201][adr-hub-extension].

## Каталог extension-параметров

В P0 отдельных флагов с источником **extension** **нет**.

| Параметр | Источник контракта | Примечание |
| -------- | ------------------ | ---------- |
| — | — | Маппинг hub-параметров (в т.ч. `scope`) — только в [mapping/](repository-selection-mapping/index.md) |

## Общие правила

### Применимость

| Условие | Действие |
| ------- | -------- |
| `upstream=github` после резолва | Extension-ключи из других upstream недопустимы ([hub][scm-repository-selection-errors]). |
| `upstream` не `github`, а в argv или профиле задан ключ только из GitLab options (например `include-inactive`) | Ошибка до старта, exit `1` ([hub][scm-repository-selection-errors]). |

### Композиция

Переносимые `--q*` и модификаторы hub (например `scope`) — по [hub][scm-repository-selection];
фильтрация по `--q*` не меняется (AND).

## Инварианты

- Таблица extension-флагов GitHub живёт **только** в этом файле (P0 — пустой каталог).
- REST и маппинг query — **не** здесь.

## Не описывать здесь

- Переносимые опции и контракт `scope` — [hub][scm-repository-selection].
- Маппинг на API — [repository-selection-mapping/](repository-selection-mapping/index.md).
- Base list и карточка — [base-enumeration.md](base-enumeration.md).
- Клиентская фильтрация — [repository-enumeration/](repository-enumeration/index.md).

<!-- LINKS -->
[scm-repository-selection]: ../../platform/scm-repository-selection.md
[scm-repository-selection-errors]: ../../platform/scm-repository-selection.md#ошибки-и-коды-выхода
[adr-hub-extension]: ../../../../ADR/ADR-26102201-scm-selection-hub-extension-specs.md
