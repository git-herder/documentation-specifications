# Маппинг `scm-search` (GitLab)

CLI → HTTP. Общий смысл флага — [hub § scm-search][scm-repository-selection-scm-search].
**Контракт argv ниже — только GitLab**; у других `upstream` свой текст в
`scm/<upstream>/repository-selection-mapping/scm-search.md`.

Upstream Search API: [GitLab Search API](https://docs.gitlab.com/api/search/).

## Когда включать

После резолва argv и профиля задан `scm-search` и `upstream=gitlab`. Иначе этот
mapping не используют.

В этой ветке **не** применяют [scope.md](scope.md) (продуктовый `--scope` hub),
[include-inactive.md](include-inactive.md), mapping `qt` / `qa` и
[enumeration/](../repository-enumeration/index.md).

**Не путать:** продуктовый флаг hub [`--scope`][scm-repository-selection-scope]
(`affiliated` / `all`) и query-параметр **`scope`** Search API (`projects`,
`issues`, …).

## Контракт argv (GitLab)

```text
--scm-search key1=val1 key2=val2 ...
```

- В argv **один** `--scm-search` (повтор — exit `1`, hub).
- После флага — одна или несколько **лексем** через **пробел**; каждая лексема —
  пара `key=value` (первый `=` отделяет key от value; в value могут быть ещё `=`).
- Лексема **без** `=` → exit `1` до старта (ошибка разбора `--scm-search`).
- Один и тот же `key` несколько раз → побеждает **последняя** пара.
- Значения с пробелами — [синтаксис опций CLI][cli-syntax] (кавычки, `--`).
- В профиле (GitLab): одна строка с теми же парами через пробел, например
  `scm-search: search=flight include_archived=true`. Символ `&` в argv и профиле
  **не** используют — только на HTTP-шаге.

## Разбор и query для Search (до HTTP)

После резолва программа **собирает** список пар «имя параметра → значение» из argv
или профиля (один ключ `scm-search` — правила hub [резолва][scm-repository-selection-resolve]).

Дальше для GitLab Search:

1. **Находит** пару с именем параметра `scope` (имя — как в HTTP, регистрозависимо).
2. Если пары `scope` **нет** — **добавляет** `scope=projects`.
3. Если `scope` **есть** и значение **без учёта регистра** не равно `projects` —
   **завершает** команду с exit `1` до HTTP (Git Herder оперирует только
   репозиториями → Search только с `scope=projects`).
4. Если `scope=projects` (любой регистр) — **нормализует** значение к `projects`.
5. **Склеивает** все пары в одну query-строку для GET: `key1=val1&key2=val2&…`,
   **кодирует** имена и значения для HTTP так же, как для query list.
6. Параметры `search`, `search_type`, `include_archived` и прочие из argv
   **не** проверяет; обязательность `search` — у GitLab API.

| Пример argv | Смысл итоговой query |
| ----------- | -------------------- |
| `--scm-search search=flight` | `search=flight` и добавленный `scope=projects` |
| `--scm-search scope=Projects search=x` | `scope=projects`, `search=x` |
| `--scm-search scope=issues search=x` | exit `1` на шаге 3 |

## HTTP Search

1. **Отправляет** на каждую страницу выборки `GET /api/v4/search` с итоговой query
   из раздела выше плюс `page` и `per_page` (как у list).
2. **Обходит** все страницы по правилам [base-enumeration § пагинация][base-enumeration-pagination]
   (offset, `X-Total-Pages`, progress bar — там же; Search API тоже offset-based
   по документации GitLab).
3. **Собирает** элементы всех страниц в один список результатов.

## Карточки репозиториев

При `scope=projects` каждый элемент ответа — объект проекта (см. Search API).

Для **каждого** элемента **наполняет** карточку по
[base-enumeration § карточка][base-enumeration-card] (как после list). Поле
`labels` — из `topics` ответа; `properties` = `{}` (enrich `qa` в этой ветке
нет).

## Ошибки HTTP

Повторы — [`--retries-number`][cli-retries], дальше [`--on-error`][cli-on-error]:

| Ситуация | Действие |
| -------- | -------- |
| Ошибка search после всех повторов | `abort` — стоп команды; `continue` — **пустой** набор репозиториев, без частичного результата |

Повторы — [cli/index.md][cli-retries].

## Не здесь

- Hub: взаимоисключение с `--q*`, продуктовый `--scope`, повтор флага.
- Контракт `--scm-search` для GitHub и других upstream.
- Реализация в `application/`.

<!-- LINKS -->
[scm-repository-selection-scm-search]: ../../../platform/scm-repository-selection.md#scm-search
[scm-repository-selection-scope]: ../../../platform/scm-repository-selection.md#scope
[scm-repository-selection-resolve]: ../../../platform/scm-repository-selection.md#резолв-с-профилем
[cli-syntax]: ../../../cli/index.md#синтаксис-опций
[base-enumeration-pagination]: ../base-enumeration.md#пагинация
[base-enumeration-card]: ../base-enumeration.md#карточка-репозитория
[cli-retries]: ../../../cli/index.md#повторы-и---on-error
[cli-on-error]: ../../../cli/index.md#--on-error
