# Базовое перечисление репозиториев (GitHub)

SSOT базового HTTP list репозиториев GitHub для [SCM-отбора][scm-repository-selection] в режиме
`--q*`. Наполнение карточки из ответа list — [scm-repository-card][scm-repository-card]; листинг CLI →
query — [repository-listing/][listing-index].

## Трассировка вверх

- [Единый отбор для массовых операций][repository-selection-capability]
- [SCM-отбор (hub)][scm-repository-selection]
- [Обобщённая карточка репозитория][scm-repository-card]
- [ADR-26102201: hub и extension-спеки SCM-отбора][adr-hub-extension]

## Назначение

GitHub-адаптер перечисляет репозитории запросами `GET /search/repositories` (относительно
API base из профиля), собирает все страницы ответа и для каждого элемента массива `items`
наполняет поля [карточки][scm-repository-card] по правилам ниже. Параметр `q` собирается по
[listing index § сборка q][listing-q-build]; участники — `qp`, `qn`, `qa` с `=`; не `qt`.
Продуктовый листинг `scope` — [listing scope][listing-scope].

## Базовый эндпоинт

|              |                                                                      |
| ------------ | -------------------------------------------------------------------- |
| Метод и путь | `GET /search/repositories`                                           |
| Назначение   | Базовый list репозиториев для отбора в режиме `--q*`                 |

Query-параметры list (кроме правил сборки `q`):

| Параметр   | Роль                                      |
| ---------- | ----------------------------------------- |
| `q`        | Поисковый запрос; сборка — только listing |
| `page`     | Номер страницы                            |
| `per_page` | Размер страницы (адаптер/профиль)         |

На один сценарий отбора все запросы списка GitHub в режиме `--q*` (кроме `--scm-search`)
опираются на этот ресурс; обогащение — [repository-enrichment/][enrichment-index].

## Ограничения API {#ограничения-api}

В силу ограничений [Search API][github-search-repositories] GitHub для этого list:

| Ограничение | Правило |
| ----------- | ------- |
| `--qp`, `--qn` | **Не** регэкспы: только строки из символов `A`–`Z`, `a`–`z`, `0`–`9`, `_`, `-`. Иначе — ошибка до HTTP, exit `1` ([hub § ошибки][scm-repository-selection-errors]). |
| Пустой `q` | Если после резолва нет фрагментов для `q` (например, только `--qt`) — ошибка до HTTP, exit `1` ([listing § сборка q][listing-q-build], hub). |
| `--scope` | **Не** влияет на query `GET /search/repositories` ([listing scope][listing-scope]). Рекомендуется задавать `--qp` с **точным** login user или organization. |

Лимит выдачи Search API — [github-search-repositories].

Клиентская фильтрация: `--qt`, `--qa` (частично) — [enumeration/][enumeration-index].
`--qp`, `--qn` — только Search `q` ([listing/][listing-index]), см.
[enumeration § назначение][enumeration-scope].

## Пагинация

GitHub для list использует **keyset-based** пагинацию через заголовок ответа `Link`
(`rel="first"`, `"prev"`, `"next"`, `"last"`). Норматив API — [Using pagination in the REST API][github-pagination-link].

### Поведение Git Herder

1. Первый запрос list с согласованным `page` (как минимум `1`) и `per_page`.
2. Из заголовка `Link` ответа **первой** страницы из URL с `rel="last"` извлекается
   query-параметр `page` — номер **последней** страницы.
3. Git Herder запрашивает страницы с `page = 1 … last`, пока не получены все элементы
   `items` list.
4. Если `rel="last"` **отсутствует**, обход продолжается по URL с `rel="next"` до
   исчерпания следующих страниц.
5. Пустая выборка: `items` отсутствует, пустой массив или `total_count` = `0` в теле
   первого ответа; дополнительные страницы не запрашиваются, если нет `rel="next"`.

## Квота запросов {#rate-limit}

В каждом ответе GitHub REST разбор квоты читает заголовки и записывает их в
[HTTP-транспорт][transport]. Транспорт сам заголовки платформы не разбирает.

| Заголовок | Значение транспорта |
| --------- | ------------------- |
| `X-Ratelimit-Limit` | Предел, целое |
| `X-Ratelimit-Remaining` | Остаток, целое |
| `X-Ratelimit-Used` | Израсходовано, целое |
| `X-Ratelimit-Reset` | Момент сброса: epoch в секундах **плюс 1 с** |

Удержание вызова и отказ от запроса — [транспорт][transport]. Какую политику
передать экземпляру и какой текст показать перед ожиданием —
[on-limit-exceeded][options-on-limit-exceeded].

## Карточка репозитория {#карточка-репозитория}

Имена и типы полей карточки — только [scm-repository-card.md][scm-repository-card]. Ниже —
наполнение из **элемента** массива `items` ответа `GET /search/repositories` (поля, которые
задаёт только этот ответ; обогащение — [repository-enrichment/][enrichment-index]).

| Карточка      | Поле элемента GitHub | Правило |
| ------------- | -------------------- | ------- |
| `uid`         | `full_name`          | См. [§ uid](#uid) |
| `name`        | `name`               | Значение поля `name` как UTF-8 string |
| `path`        | `full_name`          | `full_name` **без последнего** сегмента (`name`); сегменты через `/`. Если в `full_name` один сегмент — `path` = `""` |
| `description` | `description`        | null или отсутствие — `""` |
| `labels`      | `topics`             | Каждый элемент `topics` → одна строка в `labels`; нет `topics` или пустой массив — `[]` |
| `properties`  | —                    | Из ответа list — `{}`. При `--qa` — [enrichment qa][enrichment-qa] |

### uid {#uid}

Поле карточки `uid` — JSON **string**, значение поля `full_name` в том же элементе `items`.
Стабильный идентификатор репозитория в пределах подключения к GitHub
(см. [карточка § uid][scm-repository-card-uid]).

## Не описывать здесь

- Таблицы переносимых и extension-опций CLI — [hub][scm-repository-selection] и
  [repository-selection-options.md][github-options].
- Клиентская фильтрация `--qt`, `--qa` (и реестр «только client-side») —
  [repository-enumeration/][enumeration-index]. `--qp`, `--qn` — только листинг.
- Пошаговые HTTP-дозагрузки полей карточки —
  [repository-enrichment/][enrichment-index].
- Реализацию в `application/`.

<!-- LINKS -->
[scm-repository-selection]: ../../platform/scm-repository-selection.md
[scm-repository-selection-errors]: ../../platform/scm-repository-selection.md#ошибки-и-коды-выхода
[scm-repository-card]: ../../platform/scm-repository-card.md
[scm-repository-card-uid]: ../../platform/scm-repository-card.md#uid
[repository-selection-capability]: ../../../product-intent/capability/platform/repository-selection.md
[adr-hub-extension]: ../../../../ADR/ADR-26102201-scm-selection-hub-extension-specs.md
[listing-index]: repository-listing/index.md
[listing-q-build]: repository-listing/index.md#сборка-q
[listing-scope]: repository-listing/scope.md
[enrichment-index]: repository-enrichment/index.md
[enrichment-qa]: repository-enrichment/qa.md
[enumeration-index]: repository-enumeration/index.md
[enumeration-scope]: repository-enumeration/index.md#назначение-каталога
[github-options]: repository-selection-options.md
[options-on-limit-exceeded]: repository-selection-options.md#on-limit-exceeded
[transport]: ../../platform/scm-http-transport.md
[github-pagination-link]: https://docs.github.com/en/rest/using-the-rest-api/using-pagination-in-the-rest-api?apiVersion=2026-03-10#using-link-headers
[github-search-repositories]: https://docs.github.com/en/rest/search/search?apiVersion=2026-03-10#search-repositories
