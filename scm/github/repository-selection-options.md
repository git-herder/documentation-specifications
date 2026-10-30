# Options SCM-отбора (GitHub)

Extension-флаги SCM-отбора, имеющие смысл **только** при `upstream=github` после
резолва: контракт CLI, ключи профиля и **продуктовый эффект** без REST.

Переносимые критерии и `scope` — [hub SCM-отбора][scm-repository-selection].
Маппинг на HTTP — [repository-selection-mapping/index.md](repository-selection-mapping/index.md).
Раскладка — [ADR-26102201][adr-hub-extension].

## Каталог extension-параметров

| Имя CLI                 | Тип        | Ключ YAML            | По умолчанию |
| ----------------------- | ---------- | -------------------- | ------------ |
| `--on-limit-exceeded`   | enum       | `on-limit-exceeded`  | `await`      |

### `on-limit-exceeded` {#on-limit-exceeded}

#### Роль

Модификатор pipeline list ([base-enumeration § rate limit][base-enumeration-rate-limit]);
**не** считается критерием «задан хотя бы один» ([hub][scm-repository-selection]).

#### Допустимые значения

После резолва — только **`await`** или **`abort`** (нижний регистр). Иное — ошибка
контракта, exit `1` ([hub § ошибки][scm-repository-selection-errors]).

#### Продуктовый эффект

| После резолва   | Эффект при `X-Ratelimit-Remaining` = `0` |
| --------------- | ---------------------------------------- |
| `await` (default) | Предупреждение на консоль; ожидание до `X-Ratelimit-Reset` + 1 с; затем продолжение list ([base-enumeration][base-enumeration-rate-limit]). |
| `abort`         | Сообщение об ошибке; **без** ожидания; команда завершается с exit `1`. |

#### Область (продукт)

Режим **`--q*`**: применяется к запросам базового list
`GET /search/repositories` и связанным HTTP-шагам того же отбора, где адаптер
обрабатывает rate limit GitHub REST.

#### Профиль

Порядок резолва — [hub § резолв][scm-repository-selection-resolve] и [profiles.md][profiles].
Если флаг **явно** в argv (`--on-limit-exceeded=…`) — только argv; иначе профиль
leaf → parents → `root`, иначе default `await`.

## Общие правила

### Применимость

| Условие                                                                 | Действие                                                             |
| ----------------------------------------------------------------------- | -------------------------------------------------------------------- |
| `upstream=github` после резолва                                         | Параметры каталога применяются по справочнику.                      |
| `upstream` не `github`, а в argv или профиле задан `on-limit-exceeded` | Ошибка до старта, exit `1` ([hub][scm-repository-selection-errors]). |

### Композиция

Переносимые `--qp`, `--qn`, `--qt`, `--qa` и модификаторы действуют **вместе** (AND с
hub); см. [mapping/index.md](repository-selection-mapping/index.md).

## Инварианты

- Контракт extension-флагов GitHub живёт **только** в этом файле.
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
[base-enumeration-rate-limit]: base-enumeration.md#rate-limit
