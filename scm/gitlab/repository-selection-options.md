# Options SCM-отбора (GitLab)

Extension-флаги SCM-отбора, имеющие смысл **только** при `upstream=gitlab` после
резолва: контракт CLI, ключи профиля и **продуктовый эффект** без REST.

Переносимые критерии и `scope` — [hub SCM-отбора][scm-repository-selection].
Маппинг на HTTP — [repository-selection-mapping/index.md](repository-selection-mapping/index.md).
Раскладка — [ADR-26102201][adr-hub-extension].

## Каталог extension-параметров

| Имя CLI              | Тип        | Ключ YAML          | По умолчанию |
|----------------------|------------|--------------------|--------------|
| `--include-inactive` | логический | `include-inactive` | `false`      |

### Роль

Скрытый критерий отбора ([hub SCM-отбора][scm-repository-selection]).

### Продуктовый эффект

| После резолва     | Эффект                                                                       |
|-------------------|------------------------------------------------------------------------------|
| `false` (default) | К отборам применяется условие **только active**-репозитории.                 |
| `true`            | Условие **не** применяется (в выборку могут попасть неактивные репозитории). |

Явное `--include-inactive=false` в argv или `false` в профиле — то же, что default.

### Область (продукт)

- Режим **`--q*`**: модификатор участвует в отборе вместе с hub-критериями и
  `scope` ([mapping/include-inactive.md](repository-selection-mapping/include-inactive.md)).
- При **`--scm-search`**: правило **не** применяется ([mapping/include-inactive.md](repository-selection-mapping/include-inactive.md)).

### Профиль

Порядок резолва — [hub § резолв][scm-repository-selection-resolve] и [profiles.md][profiles].
Если флаг **явно** в argv (`--include-inactive` / `--include-inactive=false`) — только
argv; иначе профиль leaf → parents → `root`, иначе default `false`.

## Общие правила

### Применимость

| Условие                                                               | Действие                                                             |
|-----------------------------------------------------------------------|----------------------------------------------------------------------|
| `upstream=gitlab` после резолва                                       | Параметры каталога применяются по справочнику.                       |
| `upstream` не `gitlab`, а в argv или профиле задан `include-inactive` | Ошибка до старта, exit `1` ([hub][scm-repository-selection-errors]). |

### Композиция

Переносимые `--qp`, `--qn`, `--qt`, `--qa` и модификаторы действуют **вместе** (AND с
hub); см. [mapping/index.md](repository-selection-mapping/index.md).

## Инварианты

- Контракт extension-флагов GitLab живёт **только** в этом файле.
- REST — только в mapping / base-enumeration.

## Не описывать здесь

- Переносимые опции и `scope` — [hub][scm-repository-selection].
- REST — [mapping/](repository-selection-mapping/index.md) и [base-enumeration.md](base-enumeration.md).
- Клиентская фильтрация — [repository-enumeration/](repository-enumeration/index.md).

<!-- LINKS -->
[scm-repository-selection]: ../../platform/scm-repository-selection.md
[scm-repository-selection-errors]: ../../platform/scm-repository-selection.md#ошибки-и-коды-выхода
[scm-repository-selection-resolve]: ../../platform/scm-repository-selection.md#резолв-с-профилем
[profiles]: ../../platform/profiles.md
[adr-hub-extension]: ../../../../ADR/ADR-26102201-scm-selection-hub-extension-specs.md
