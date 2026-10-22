# Расширения SCM-отбора (GitLab)

Контракт **GitLab-специфичных** опций SCM-отбора: флаги команд и ключи YAML-профиля,
имеющие смысл только при `upstream=gitlab` после резолва.

Переносимые критерии (`--qp`, `--qn`, `--qt`, `--qa`, `--scm-search`, `--scope`) —
[hub SCM-отбора][scm-repository-selection]. Раскладка hub / extension /
enumeration — [ADR-26102201][adr-hub-extension]. HTTP и пагинация —
[repository-enumeration.md](repository-enumeration.md).

## Трассировка вверх

- [SCM-отбор (hub)][scm-repository-selection]
- [ADR-26102201: hub и extension-спеки SCM-отбора][adr-hub-extension]
- [Профили и выбор значений][profiles]

## Применимость

| Условие | Действие |
| ------- | -------- |
| `upstream=gitlab` после резолва | Ключи этой спеки допустимы. |
| `upstream=github` (или иной), а в argv или профиле задан ключ из таблицы ниже | Ошибка до старта, exit `1` ([hub][scm-repository-selection#ошибки-и-коды-выхода]). |

## Таблица опций

| Имя | Тип | По умолчанию | Назначение |
| --- | --- | ------------ | ---------- |
| `--include-inactive` | логический | `false` | Включать репозитории **не** в статусе active (запланированные к удалению и аналоги). |

Ключ в YAML-профиле: `include-inactive` (без `--`).

## Резолв с профилем

Порядок и правило «один ключ — одно происхождение» — как для переносимых ключей в
[hub][scm-repository-selection#резолв-с-профилем] и [profiles.md][profiles]; здесь **не**
повторяются.

| Ситуация | Итог для `include-inactive` |
| -------- | ----------------------------- |
| Флаг **явно** в argv (`--include-inactive`, `--include-inactive=false`) | Только argv. |
| Флаг в argv **не** задан | Значение из профиля по цепочке leaf → parents → `root`; иначе default `false`. |

## Семантика `include-inactive`

Параметр — **модификатор** pipeline отбора, **не** критерий: один только
`include-inactive` **не** удовлетворяет проверку «задан хотя бы один критерий» в
[hub][scm-repository-selection#ошибки-и-коды-выхода].

| После резолва | Поведение |
| ------------- | --------- |
| `false` (default) | Ко **всем** HTTP-запросам GitLab-адаптера в рамках **одной** операции SCM-отбора (перечисление, дозагрузка метаданных для `qt` / `qa` и прочие шаги отбора) добавляется продуктовое условие **`active=true`**. Маппинг на параметры REST API — в [repository-enumeration.md](repository-enumeration.md), не здесь. |
| `true` | Условие `active=true` **не** добавляется (в выборку могут попасть неактивные репозитории). |

### Ветка `--scm-search`

Если после резолва задан `--scm-search`, правило `active=true` **не** применяется
(по аналогии с [scope в hub][scm-repository-selection-scope]).

## Композиция с переносимыми критериями

Условия `--qp`, `--qn`, `--qt`, `--qa` и модификатор `include-inactive` действуют
**вместе** в одном отборе; логика фильтрации по `--q*` не меняется.

## Инварианты

- Таблица опций GitLab extension живёт **только** в этом файле.
- `include-inactive` не является критерием отбора.
- В режиме `--scm-search` фильтр `active=true` **не** применяется.
- При `include-inactive=false` фильтр `active=true` на всех запросах GitLab в операции
  отбора, кроме ветки `--scm-search`.

## Не описывать здесь

- Переносимые флаги `--qp` … `--scope`.
- REST-эндпоинты и поля ответа — [repository-enumeration.md](repository-enumeration.md).
- Реализацию в `application/`.

<!-- LINKS -->
[scm-repository-selection]: ../../platform/scm-repository-selection.md
[scm-repository-selection#ошибки-и-коды-выхода]: ../../platform/scm-repository-selection.md#ошибки-и-коды-выхода
[scm-repository-selection#резолв-с-профилем]: ../../platform/scm-repository-selection.md#резолв-с-профилем
[profiles]: ../../platform/profiles.md
[adr-hub-extension]: ../../../ADR/ADR-26102201-scm-selection-hub-extension-specs.md
[scm-repository-selection-scope]: ../../platform/scm-repository-selection.md#scope
