# Расширения SCM-отбора (GitLab)

Параметры SCM-отбора, которые на **GitLab** задаются отдельными флагами extension или
требуют **реализации** переносимого `--scope` на API. Контракт переносимых критериев —
[hub SCM-отбора][scm-repository-selection]. Раскладка hub / extension / enumeration —
[ADR-26102201][adr-hub-extension]. HTTP — [repository-enumeration.md](repository-enumeration.md).

**Источник контракта:** `hub` — имя и значения задаются в hub; `extension` — флаг
существует только для этого `upstream`.

## Каталог параметров (2)

| Параметр | Источник контракта | Ключ YAML | Роль | Кратко |
| -------- | ------------------ | --------- | ---- | ------ |
| `scope` | hub | `scope` | модификатор pipeline | На API: **membership** при `affiliated`; см. [§ scope](#scope) |
| `include-inactive` | extension | `include-inactive` | модификатор pipeline | По умолчанию только **active**-репозитории; см. [§ include-inactive](#include-inactive) |

## Справочник параметров

### `scope` — реализация на GitLab {#scope}

#### Контракт

Значения `affiliated` \| `all`, default `affiliated`, ключ `scope` — только
[hub][scm-repository-selection-scope]. Здесь **не** дублируются.

#### Роль

**Модификатор** pipeline отбора; один только `scope` **не** считается критерием
отбора ([hub][scm-repository-selection-errors]).

#### Область применения

- Режим **`--q*`**, не `--scm-search`: ко **всем** HTTP-запросам GitLab-адаптера в
  рамках **одной** операции SCM-отбора (перечисление, дозагрузка для `qt` / `qa` и
  прочие шаги отбора).
- При **`--scm-search`**: `scope` к запросу в SCM **не** применяется
  ([hub][scm-repository-selection-scope]).

#### Поведение по значениям

| `scope` после резолва | Действие адаптера GitLab |
| --------------------- | ------------------------ |
| `affiliated` | Добавляется ограничение перечисления в продуктовой семантике **membership** (репозитории, связанные с текущим пользователем). Имена параметров REST — в [repository-enumeration.md](repository-enumeration.md). |
| `all` | Ограничение **membership** / affiliation **не** добавляется. |

На **любом** хосте GitLab, где адаптер умеет применить ограничение, `affiliated`
использует ту же продуктовую семантику.

#### Условия хоста

На **`gitlab.com`** — [ADR-26102002][adr-saas-guard], раздел
[scope на публичном SaaS][adr-saas-guard-scope] (единый SSOT для `github.com` и
`gitlab.com`). На прочих хостах этот раздел ADR **не** применяется; для `affiliated` /
`all` действует только таблица «Поведение по значениям» выше.

#### Профиль

Резолв `scope` — как для переносимых ключей в [hub][scm-repository-selection-resolve]
и [profiles.md][profiles].

---

### `include-inactive` {#include-inactive}

#### Контракт

| Имя CLI | Тип | По умолчанию |
| ------- | --- | ------------ |
| `--include-inactive` | логический | `false` |

Ключ YAML: `include-inactive` (без `--`).

#### Роль

**Модификатор** pipeline; **не** критерий отбора ([hub][scm-repository-selection-errors]).

#### Область применения

- Все HTTP-запросы GitLab-адаптера в рамках **одной** операции SCM-отбора (как у
  [scope](#scope) в режиме `--q*`).
- При **`--scm-search`**: правило `active=true` **не** применяется.

#### Поведение по значениям

| После резолва | Действие |
| ------------- | -------- |
| `false` (default) | К запросам добавляется продуктовое условие **`active=true`**. Маппинг на REST — в [repository-enumeration.md](repository-enumeration.md). |
| `true` | Условие `active=true` **не** добавляется (в выборку могут попасть неактивные репозитории). |

Явное `--include-inactive=false` в argv или `false` в профиле — то же, что default.

#### Условия хоста

Не применимо (правило не зависит от `gitlab.com` / self-hosted).

#### Профиль

Порядок резолва — [hub][scm-repository-selection-resolve] и [profiles.md][profiles].
Для `include-inactive`: если флаг **явно** в argv (`--include-inactive` /
`--include-inactive=false`) — только argv; иначе профиль leaf → parents → `root`, иначе
default `false`.

## Общее для всех параметров extension

### Применимость

| Условие | Действие |
| ------- | -------- |
| `upstream=gitlab` после резолва | Параметры каталога применяются по справочнику. |
| `upstream` не `gitlab`, а в argv или профиле задан ключ только из этой спеки (например `include-inactive`) | Ошибка до старта, exit `1` ([hub][scm-repository-selection-errors]). |

### Композиция

Переносимые `--qp`, `--qn`, `--qt`, `--qa` и модификаторы из каталога действуют
**вместе**; фильтрация по `--q*` не меняется (AND с hub).

### Трассировка вверх

- [SCM-отбор (hub)][scm-repository-selection]
- [ADR-26102002][adr-saas-guard]
- [ADR-26102201][adr-hub-extension]
- [Профили][profiles]

## Инварианты

- Каталог и справочник GitLab extension живут **только** в этом файле.
- Оба параметра каталога — модификаторы, не критерии отбора.
- В ветке `--scm-search` не применяются ни `scope`, ни `active=true` от
  `include-inactive`.

## Не описывать здесь

- Таблицу переносимых опций `--qp` … `--scm-search` и контракт значений `scope` —
  [hub][scm-repository-selection].
- REST-эндпоинты — [repository-enumeration.md](repository-enumeration.md).
- Реализацию в `application/`.

<!-- LINKS -->
[scm-repository-selection]: ../../platform/scm-repository-selection.md
[scm-repository-selection-scope]: ../../platform/scm-repository-selection.md#scope
[scm-repository-selection-errors]: ../../platform/scm-repository-selection.md#ошибки-и-коды-выхода
[scm-repository-selection-resolve]: ../../platform/scm-repository-selection.md#резолв-с-профилем
[profiles]: ../../platform/profiles.md
[adr-hub-extension]: ../../../ADR/ADR-26102201-scm-selection-hub-extension-specs.md
[adr-saas-guard]: ../../../ADR/ADR-26102002-public-saas-scm-affiliation-guard.md
[adr-saas-guard-scope]: ../../../ADR/ADR-26102002-public-saas-scm-affiliation-guard.md#scope-public-saas
