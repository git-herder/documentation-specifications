# Листинг `scm-search` (GitHub)

CLI → HTTP. Общий смысл флага — [hub § scm-search][scm-repository-selection-scm-search].
**Контракт argv ниже — только GitHub**; у других `upstream` свой текст в
`scm/<upstream>/repository-listing/scm-search.md`.

Upstream Search API: [Search repositories][github-search-repositories].

## Когда включать

После резолва argv и профиля задан `scm-search` и `upstream=github`. Иначе этот
листинг не используют.

В этой ветке **не** применяют [scope.md](scope.md), сборку `q` из `--q*`
([index § сборка q][listing-q-build]), listing `qt` / `qa`,
[enumeration/](../repository-enumeration/index.md) и
[enrichment/](../repository-enrichment/index.md).

Продуктовый [`--scope`][scm-repository-selection-scope] на query **не** влияет.
[`--on-limit-exceeded`][options-on-limit-exceeded] query **не** меняет: это
политика квоты [транспорта][transport] на HTTP этой ветки, не фрагмент `q`.

## Контракт argv (GitHub)

```text
--scm-search key1=val1 key2=val2 ...
```

- В argv **один** `--scm-search` (повтор — exit `1`, hub).
- После флага — одна или несколько **лексем** через **пробел**; каждая лексема —
  пара `key=value` (первый `=` отделяет key от value; в value могут быть ещё `=`).
- Лексема **без** `=` → exit `1` до старта (ошибка разбора `--scm-search`).
- Один и тот же `key` несколько раз → побеждает **последняя** пара.
- Значения с пробелами — [синтаксис опций CLI][cli-syntax] (кавычки, `--`).
- В профиле (GitHub): одна строка с теми же парами через пробел, например
  `scm-search: q=org:acme language:go`. Символ `&` в argv и профиле **не**
  используют — только на HTTP-шаге.

Обязательная пара — `q` (имя регистрозависимо). Нет пары `q` или value пустое
после trim → exit `1` до HTTP. Остальные пары (`sort`, `order` и прочие) продукт
**не** проверяет; их принимает или отвергает GitHub API.

## Разбор и query для Search (до HTTP)

После резолва программа **собирает** список пар «имя параметра → значение» из argv
или профиля (один ключ `scm-search` — правила hub
[резолва][scm-repository-selection-resolve]).

1. Проверяет пару `q` (см. выше).
2. **Склеивает** все пары в одну query-строку для GET: `key1=val1&key2=val2&…`,
   **кодирует** имена и значения для HTTP так же, как для query list.
3. К каждой странице добавляет `page` и `per_page` (размер страницы — как у
   [базового list][base-enumeration], здесь не дублируется).

Сборка `q` из `--qp` / `--qn` / `--qa` в этой ветке **не** выполняется.

| Пример argv | Смысл итоговой query |
| ----------- | -------------------- |
| `--scm-search q=org:acme` | `q=org:acme` плюс `page` и `per_page` |
| `--scm-search q=org:acme sort=updated` | `q` и `sort` как заданы |
| `--scm-search sort=stars` | exit `1`: нет `q` |

## HTTP Search

1. **Отправляет** на каждую страницу выборки `GET /search/repositories` с итоговой
   query из раздела выше.
2. **Обходит** страницы по правилам [base-enumeration § пагинация][base-enumeration-pagination]
   (заголовок `Link`). Прогресс-бар — по страницам, как в
   [оркестрации][scm-index].
3. **Собирает** элементы `items` всех страниц в один список результатов.

Лимит выдачи Search API — поведение GitHub;
[документация search/repositories][github-search-repositories].

## Карточки репозиториев

Для **каждого** элемента `items` **наполняет** карточку по
[base-enumeration § карточка][base-enumeration-card], включая `url` из
`html_url`. Поле `properties` = `{}` (ветка `scm-search` не использует
[enrichment/qa.md](../repository-enrichment/qa.md)).

## Ошибки HTTP

Повторы — [`--retries-number`][cli-retries], дальше [`--on-error`][cli-on-error]:

| Ситуация | Действие |
| -------- | -------- |
| Ошибка search после всех повторов | `abort` — стоп команды; `continue` — **пустой** набор репозиториев, без частичного результата |

Невалидный `url` в элементе — [карточка § url][scm-repository-card-url].

## Не здесь

- Hub: взаимоисключение с `--q*`, продуктовый `--scope`, повтор флага.
- Сборка `q` режима `--q*` — [index § сборка q][listing-q-build].
- Контракт `--scm-search` для GitLab и других upstream.
- Реализация в `application/`.

<!-- LINKS -->
[scm-repository-selection-scm-search]: ../../../platform/scm-repository-selection.md#scm-search
[scm-repository-selection-scope]: ../../../platform/scm-repository-selection.md#scope
[scm-repository-selection-resolve]: ../../../platform/scm-repository-selection.md#резолв-с-профилем
[scm-repository-card-url]: ../../../platform/scm-repository-card.md#url
[options-on-limit-exceeded]: ../repository-selection-options.md#on-limit-exceeded
[transport]: ../../../platform/scm-http-transport.md
[cli-syntax]: ../../../cli/index.md#синтаксис-опций
[listing-q-build]: index.md#сборка-q
[base-enumeration]: ../base-enumeration.md
[base-enumeration-pagination]: ../base-enumeration.md#пагинация
[base-enumeration-card]: ../base-enumeration.md#карточка-репозитория
[scm-index]: ../../index.md
[cli-retries]: ../../../cli/index.md#повторы-и---on-error
[cli-on-error]: ../../../cli/index.md#--on-error
[github-search-repositories]: https://docs.github.com/en/rest/search/search?apiVersion=2026-03-10#search-repositories
