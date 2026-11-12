# Options SCM-отбора (GitHub)

Extension-флаги SCM-отбора, имеющие смысл **только** при `upstream=github` после
резолва: контракт CLI, ключи профиля и **продуктовый эффект** без REST.

Переносимые критерии и `scope` — [hub SCM-отбора][scm-repository-selection].
Листинг на HTTP — [repository-listing/index.md](repository-listing/index.md).
Раскладка — [ADR-26102201][adr-hub-extension].

## Каталог extension-параметров

| Имя CLI                 | Тип        | Ключ YAML            | По умолчанию |
| ----------------------- | ---------- | -------------------- | ------------ |
| `--on-limit-exceeded`   | enum       | `on-limit-exceeded`  | `await`      |

### `on-limit-exceeded` {#on-limit-exceeded}

#### Роль

Модификатор квоты общего [HTTP-транспорта][transport] при `upstream=github`
([соответствие заголовков][base-enumeration-rate-limit]).
Не считается критерием «задан хотя бы один» ([hub][scm-repository-selection]).

#### Допустимые значения

После резолва — только **`await`** или **`abort`** (нижний регистр). Иное — ошибка
контракта, exit `1` ([hub § ошибки][scm-repository-selection-errors]).

#### Продуктовый эффект

| После резолва | Эффект при нулевом остатке |
| ------------- | -------------------------- |
| `await` (по умолчанию) | Предупреждение на консоль о достижении квоты и ожидании до момента сброса; транспорт удерживает вызов. Момент сброса — [квота GitHub][base-enumeration-rate-limit]. |
| `abort` | Сообщение об ошибке; запрос не выполняется; команда завершается с exit `1`. |

#### Область (продукт)

Режим **`--q*`**: политика общего экземпляра транспорта на все HTTP-запросы
этого отбора (перечисление и обогащение), не только базовый list.

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
hub); см. [listing/index.md](repository-listing/index.md).

## Инварианты

- Контракт extension-флагов GitHub живёт **только** в этом файле.
- Листинг и обогащение — только в listing / enrichment / base-enumeration.

## Не описывать здесь

- Переносимые опции и `scope` — [hub][scm-repository-selection].
- Листинг и обогащение — [listing/](repository-listing/index.md),
  [enrichment/](repository-enrichment/index.md) и [base-enumeration.md](base-enumeration.md).
- Клиентская фильтрация — [repository-enumeration/](repository-enumeration/index.md).
- Удержание вызова, отказ от запроса и хранение квоты —
  [HTTP-транспорт][transport]. Текст предупреждения — в этом файле.

<!-- LINKS -->
[scm-repository-selection]: ../../platform/scm-repository-selection.md
[scm-repository-selection-errors]: ../../platform/scm-repository-selection.md#ошибки-и-коды-выхода
[scm-repository-selection-resolve]: ../../platform/scm-repository-selection.md#резолв-с-профилем
[profiles]: ../../platform/profiles.md
[adr-hub-extension]: ../../../../ADR/ADR-26102201-scm-selection-hub-extension-specs.md
[base-enumeration-rate-limit]: base-enumeration.md#rate-limit
[transport]: ../../platform/scm-http-transport.md
