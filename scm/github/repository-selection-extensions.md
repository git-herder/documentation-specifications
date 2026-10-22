# Расширения SCM-отбора (GitHub)

Параметры SCM-отбора, которые на **GitHub** задаются отдельными флагами extension или
требуют **реализации** переносимого `--scope` на API. Контракт переносимых критериев —
[hub SCM-отбора][scm-repository-selection]. Раскладка hub / extension / enumeration —
[ADR-26102201][adr-hub-extension]. HTTP — [repository-enumeration.md](repository-enumeration.md).

**Источник контракта:** `hub` — имя и значения задаются в hub; `extension` — флаг
существует только для этого `upstream`.

## Каталог параметров (1)

| Параметр | Источник контракта | Ключ YAML | Роль | Кратко |
| -------- | ------------------ | --------- | ---- | ------ |
| `scope` | hub | `scope` | модификатор pipeline | На API: **involves** при `affiliated`; см. [§ scope](#scope) |

В P0 отдельных флагов с источником `extension` **нет**.

## Справочник параметров

### `scope` — реализация на GitHub {#scope}

#### Контракт

Значения `affiliated` \| `all`, default `affiliated`, ключ `scope` — только
[hub][scm-repository-selection-scope]. Здесь **не** дублируются.

#### Роль

**Модификатор** pipeline отбора; один только `scope` **не** считается критерием
отбора ([hub][scm-repository-selection-errors]).

#### Область применения

- Режим **`--q*`**, не `--scm-search`: ко **всем** HTTP-запросам GitHub-адаптера в
  рамках **одной** операции SCM-отбора (перечисление, дозагрузка для `qt` / `qa` и
  прочие шаги отбора).
- При **`--scm-search`**: `scope` к запросу в SCM **не** применяется
  ([hub][scm-repository-selection-scope]).

#### Поведение по значениям

| `scope` после резолва | Действие адаптера GitHub |
| --------------------- | ------------------------ |
| `affiliated` | Добавляется ограничение перечисления в продуктовой семантике **involves** (репозитории, связанные с текущим пользователем). Имена параметров REST — в [repository-enumeration.md](repository-enumeration.md). |
| `all` | Ограничение **involves** / affiliation **не** добавляется. |

На **любом** хосте GitHub, где адаптер умеет применить ограничение, `affiliated`
использует ту же продуктовую семантику.

#### Условия хоста

На **`github.com`** — [ADR-26102002][adr-saas-guard], раздел
[scope на публичном SaaS][adr-saas-guard-scope] (единый SSOT для `github.com` и
`gitlab.com`). На прочих хостах этот раздел ADR **не** применяется; для `affiliated` /
`all` действует только таблица «Поведение по значениям» выше.

#### Профиль

Резолв `scope` — как для переносимых ключей в [hub][scm-repository-selection-resolve]
и [profiles.md][profiles].

## Общее для всех параметров extension

### Применимость

| Условие | Действие |
| ------- | -------- |
| `upstream=github` после резолва | Параметры каталога применяются по справочнику. |
| `upstream` не `github`, а заданы правила этой спеки как для чужого extension | Ошибка до старта, exit `1` ([hub][scm-repository-selection-errors]). |

### Композиция

Переносимые `--q*` и модификатор `scope` действуют **вместе**; фильтрация по `--q*`
не меняется (AND с hub).

### Трассировка вверх

- [SCM-отбор (hub)][scm-repository-selection]
- [ADR-26102002][adr-saas-guard]
- [ADR-26102201][adr-hub-extension]
- [Профили][profiles]

## Инварианты

- Каталог и справочник GitHub extension живут **только** в этом файле.
- `scope` — модификатор, не критерий отбора.
- В ветке `--scm-search` `scope` к запросу **не** применяется.

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
