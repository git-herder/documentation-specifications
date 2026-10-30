# Базовое перечисление репозиториев (GitHub)

SSOT базового HTTP list репозиториев GitHub для [SCM-отбора][scm-repository-selection] в режиме
`--q*`. Карточка после list — [scm-repository-card][scm-repository-card]; маппинг CLI →
query — [repository-selection-mapping/][mapping-index].

## Трассировка вверх

- [Единый отбор для массовых операций][repository-selection-capability]
- [SCM-отбор (hub)][scm-repository-selection]
- [Обобщённая карточка репозитория][scm-repository-card]
- [ADR-26102201: hub и extension-спеки SCM-отбора][adr-hub-extension]

## Назначение

GitHub-адаптер перечисляет репозитории запросами `GET /search/repositories` (относительно
API base из профиля), собирает все страницы ответа и для каждого элемента массива `items`
наполняет поля [карточки][scm-repository-card] по правилам ниже. Параметр `q` собирается по
[mapping qp][mapping-qp] и [mapping qn][mapping-qn]; продуктовый маппинг `scope` — [mapping scope][mapping-scope].

## Базовый эндпоинт

|              |                                                                      |
| ------------ | -------------------------------------------------------------------- |
| Метод и путь | `GET /search/repositories`                                           |
| Назначение   | Базовый list репозиториев для отбора в режиме `--q*`                 |

Query-параметры list (кроме правил сборки `q`):

| Параметр   | Роль                                      |
| ---------- | ----------------------------------------- |
| `q`        | Поисковый запрос; сборка — только mapping |
| `page`     | Номер страницы                            |
| `per_page` | Размер страницы (адаптер/профиль)         |

На один сценарий отбора все запросы списка GitHub в режиме `--q*` (кроме `--scm-search`)
опираются на этот ресурс; дозагрузка и прочие пути — [mapping/][mapping-index].

## Ограничения API {#ограничения-api}

В силу ограничений [Search API][github-search-repositories] GitHub для этого list:

| Ограничение | Правило |
| ----------- | ------- |
| `--qp`, `--qn` | **Не** регэкспы: только строки из символов `A`–`Z`, `a`–`z`, `0`–`9`, `_`, `-`. Иначе — ошибка до HTTP, exit `1` ([hub § ошибки][scm-repository-selection-errors]). |
| `--scope` | **Не** влияет на query `GET /search/repositories` ([mapping scope][mapping-scope]). Рекомендуется задавать `--qp` с **точным** login user или organization. |

Клиентская проверка `qp` / `qn` после list — [repository-enumeration/][enumeration-index] (без регэкп).

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

## Rate limit {#rate-limit}

В каждом ответе GitHub REST адаптер учитывает заголовки:

| Заголовок | Семантика |
| --------- | --------- |
| `X-Ratelimit-Limit` | Лимит запросов в текущем окне (целое) |
| `X-Ratelimit-Remaining` | Оставшиеся запросы в окне (целое) |
| `X-Ratelimit-Used` | Уже использовано в окне (целое) |
| `X-Ratelimit-Reset` | Epoch (секунды), момент сброса текущего окна |

При `X-Ratelimit-Remaining` = `0` после очередного ответа:

1. Если после резолва [on-limit-exceeded][options-on-limit-exceeded] = `await` (default):
   вывести на консоль предупреждение о достижении лимита и ожидании до
   `X-Ratelimit-Reset`; ждать до момента `X-Ratelimit-Reset` + 1 с, затем продолжить
   list.
2. Если `abort` — только [options § on-limit-exceeded][options-on-limit-exceeded].

## Карточка репозитория {#карточка-репозитория}

Имена и типы полей карточки — только [scm-repository-card.md][scm-repository-card]. Ниже —
наполнение из **элемента** массива `items` ответа `GET /search/repositories` (до mapping-enrich).

| Карточка      | Поле элемента GitHub | Правило |
| ------------- | -------------------- | ------- |
| `uid`         | `full_name`          | См. [§ uid](#uid) |
| `name`        | `name`               | Значение поля `name` как UTF-8 string |
| `path`        | `full_name`          | `full_name` **без последнего** сегмента (`name`); сегменты через `/`. Если в `full_name` один сегмент — `path` = `""` |
| `description` | `description`        | null или отсутствие — `""` |
| `labels`      | `topics`             | Каждый элемент `topics` → одна строка в `labels`; нет `topics` или пустой массив — `[]` |
| `properties`  | —                    | На шаге только base list — `{}`. Наполнение при `--qa` — mapping `qa` (когда спека появится) |

### uid {#uid}

Поле карточки `uid` — JSON **string**, значение поля `full_name` в том же элементе `items`.
Стабильный идентификатор репозитория в пределах подключения к GitHub
(см. [карточка § uid][scm-repository-card-uid]).

## Не описывать здесь

- Таблицы переносимых и extension-опций CLI — [hub][scm-repository-selection] и
  [repository-selection-options.md][github-options].
- Клиентская фильтрация `--qp`, `--qn`, `--qt`, `--qa` —
  [repository-enumeration/][enumeration-index].
- Пошаговые HTTP enrich (кроме однострочных отсылок выше) — [mapping/][mapping-index].
- Реализацию в `application/`.

<!-- LINKS -->
[scm-repository-selection]: ../../platform/scm-repository-selection.md
[scm-repository-selection-errors]: ../../platform/scm-repository-selection.md#ошибки-и-коды-выхода
[scm-repository-card]: ../../platform/scm-repository-card.md
[scm-repository-card-uid]: ../../platform/scm-repository-card.md#uid
[repository-selection-capability]: ../../../product-intent/capability/platform/repository-selection.md
[adr-hub-extension]: ../../../../ADR/ADR-26102201-scm-selection-hub-extension-specs.md
[mapping-index]: repository-selection-mapping/index.md
[mapping-qp]: repository-selection-mapping/qp.md
[mapping-qn]: repository-selection-mapping/qn.md
[mapping-scope]: repository-selection-mapping/scope.md
[enumeration-index]: repository-enumeration/index.md
[github-options]: repository-selection-options.md
[options-on-limit-exceeded]: repository-selection-options.md#on-limit-exceeded
[github-pagination-link]: https://docs.github.com/en/rest/using-the-rest-api/using-pagination-in-the-rest-api?apiVersion=2026-03-10#using-link-headers
[github-search-repositories]: https://docs.github.com/en/rest/search/search?apiVersion=2026-03-10#search-repositories
