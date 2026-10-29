# Базовое перечисление репозиториев (GitLab)

SSOT базового HTTP list проектов GitLab для [SCM-отбора](../../platform/scm-repository-selection.md) в режиме
`--q*`. Карточка после list — [scm-repository-card](../../platform/scm-repository-card.md); маппинг CLI →
query — [repository-selection-mapping/](repository-selection-mapping/index.md).

Отдельная ветка API для `--scm-search` — [scm-search.md](repository-selection-mapping/scm-search.md),
не этот list.

## Трассировка вверх

- [Единый отбор для массовых операций](../../../product-intent/capability/platform/repository-selection.md)
- [SCM-отбор (hub)](../../platform/scm-repository-selection.md)
- [Обобщённая карточка репозитория](../../platform/scm-repository-card.md)
- [ADR-26102201: hub и extension-спеки SCM-отбора](../../../../ADR/ADR-26102201-scm-selection-hub-extension-specs.md)

## Назначение

GitLab-адаптер перечисляет репозитории запросами `GET /api/v4/projects` (относительно
base URL инстанса из профиля), собирает все страницы ответа и для каждого элемента JSON
наполняет поля [карточки](../../platform/scm-repository-card.md) по правилам ниже. Дополнительные query на
тот же list для `scope` и `include-inactive` — [§ membership](#query-membership-scope) и
[§ active](#query-active-filter); продуктовый маппинг значений CLI — только в
[mapping/](repository-selection-mapping/index.md).

## Базовый эндпоинт

|              |                                                                 |
| ------------ | --------------------------------------------------------------- |
| Метод и путь | `GET /api/v4/projects`                                          |
| Назначение   | Базовый list репозиториев (проектов) для отбора в режиме `--q*` |

На один сценарий отбора все list-запросы GitLab в режиме `--q*` (кроме `--scm-search`)
опираются на этот ресурс; enrich и иные пути — [mapping/](repository-selection-mapping/index.md).

## Пагинация

GitLab для list всегда использует **offset-based** пагинацию: параметры запроса и заголовки
ответа.

| Роль | Имя | Семантика |
| ---- | --- | --------- |
| Query | `page` | Номер страницы, которую нужно вернуть |
| Query | `per_page` | Количество элементов на странице |
| Response header | `X-Total` | Всего элементов в выборке; `0` или отсутствие/null означает, что нет подходящих элементов |
| Response header | `X-Total-Pages` | Всего страниц при текущем значении `per_page` |
| Response header | `X-Next-Page` | Номер следующей страницы; null или `0` — текущая страница последняя |

### Поведение Git Herder

1. Первый запрос — как минимум `page=1` и согласованный `per_page` (размер страницы задаёт

   адаптер/профиль, не дублируется здесь).
2. Из ответа **первой** страницы читается `X-Total-Pages`.
3. Git Herder запрашивает страницы `page = 2 … X-Total-Pages`, пока не получены все

   элементы list.
4. Пустая выборка: `X-Total` равен `0` или отсутствует/null; допустима одна страница с

   пустым телом JSON-массива. Дополнительные страницы не запрашиваются, если
    `X-Total-Pages` отсутствует или равен `1` при пустом теле первой страницы.

Заголовок `X-Next-Page` информативен; нормативный обход страниц — по `X-Total-Pages`
после первого ответа.

#### Прогресс при обходе list

При обходе страниц Git Herder показывает **progress bar**. Значение **`X-Total`**
из ответа **первой** страницы — шкала **100%** (всего элементов в выборке). Текущий
прогресс — число уже учтённых элементов list, вычисляемое по номеру обработанной
страницы `page` и `per_page` этого запроса:

```text
current = min(page × per_page, X-Total)
```

Доля progress bar: `current / X-Total`. При `X-Total` = `0` или отсутствии/null шкала
100% не задаётся, текущий прогресс — `0`. После последней страницы `current` равен
`X-Total`.

## Карточка репозитория

Имена и типы полей карточки — только [scm-repository-card.md](../../platform/scm-repository-card.md). Ниже —
наполнение из **элемента** JSON-массива ответа `GET /api/v4/projects` (до mapping-enrich).

| Карточка      | Поле проекта GitLab   | Правило                                                                                                                                                                         |
| ------------- | --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `uid`         | `id`                  | См. [§ uid](#uid)                                                                                                                                                               |
| `name`        | `name`                | Значение поля `name` из API как UTF-8 string                                                                                                                                    |
| `path`        | `path_with_namespace` | Значение карточки `path` = `path_with_namespace` **без последнего** сегмента (slug/path проекта); сегменты через `/`. Если в `path_with_namespace` один сегмент — `path` = `""` |
| `description` | `description`         | Если в API null или поле отсутствует — `""`                                                                                                                                     |
| `labels`      | `topics`              | Каждый элемент массива `topics` → одна строка в `labels`; нет `topics` или пустой массив — `[]`                                                                                 |
| `properties`  | —                     | На шаге только base list — `{}`. Наполнение из `custom_properties` — REST в [mapping/qa.md](repository-selection-mapping/qa.md)                                                 |

### uid {#uid}

Поле карточки `uid` — JSON **number**, значение поля `id` проекта GitLab в том же
элементе list. Стабильный идентификатор репозитория в пределах подключения к инстансу GitLab
(см. [карточка § uid](../../platform/scm-repository-card.md#uid)).

## Не описывать здесь

- Таблицы переносимых и extension-опций CLI — [hub](../../platform/scm-repository-selection.md) и
[repository-selection-options.md](repository-selection-options.md).
- Клиентская фильтрация `--qp`, `--qn`, `--qt`, `--qa` —
[repository-enumeration/](repository-enumeration/index.md).
- Пошаговые HTTP enrich (кроме однострочных отсылок выше) — [mapping/](repository-selection-mapping/index.md).
- Реализацию в `application/`.
