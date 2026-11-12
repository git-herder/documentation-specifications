# Загрузка из SCM и обогащение карточек репозиториев

SSOT **оркестрации** процесса ArchiMate «Загрузка из SCM и обогащение карточек
репозиториев» (`id-649305ae`). Переиспользуется командами, которым нужен набор
[карточек репозитория][card] из SCM.

Критерии отбора, резолв argv и профиля — [hub SCM-отбора][hub].
HTTP-контракты листинга и обогащения — для каждого `upstream` ниже; клиентская
фильтрация — [repository-enumeration/][enumeration-github].

## Трассировка вверх

- [Единый отбор для массовых операций][repository-selection]
- Навигация по модели: [architecture/README.md][arch-readme];
  артефакт [git-herder.archimate][archimate]
- [ADR-26102201][adr-hub]
  (раскладка hub / listing / enrichment / enumeration)
- Компоновка кода: [ADR-26110801][adr-layout]
- Повторы объектов прогона — [очередь повторов][queue]

## Трассировка ArchiMate (приложение SCM) {#archimate-scm-application}

View **SCM Listing**, view **SCM Enrichment** и процесс `id-649305ae`
(см. **Report command flow** в модели) описывают один контур: сначала сборка
клиентов, затем HTTP через готовые интерфейсы.

| Фаза | ArchiMate (интерфейсы / сервисы) | Шаг в процессе `id-649305ae` |
| ---- | -------------------------------- | ---------------------------- |
| Сборка листинга | `ScmListingFactory`, `ScmListingBuilder`, сервис «Создание listing-клиента» | «Собрать REST-клиент server-side отборов для текущего upstream» |
| Сборка обогащения | `ScmEnrichmentFactory`, сервис «Создание enrichment'ов» | «Собрать массив заданий обогащения для получения данных и записи в карточку» |
| Листинг | **ScmListing** | «Выполнить парсинг страниц…» |
| Обогащение | **ScmEnrichment**, порт записи карточки | «Выполнить обогащение карточек» |

`ScmListingBuilder` только настраивает запрос листинга перед сборкой клиента.
У обогащения сборщика запроса нет: `ScmEnrichmentFactory` по платформе и
критериям отбора возвращает готовые реализации `ScmEnrichment`. Пагинация
вызывает **ScmListing**. Обогащение вызывает приём задания у карточки.
Навигация по модели — [architecture/README.md][arch-readme].

## Слои per upstream

| Слой | Путь |
| ---- | ---- |
| Hub | [platform/scm-repository-selection.md][hub] |
| Options | `scm/<upstream>/repository-selection-options.md` |
| Base list / карточка из list | `scm/<upstream>/base-enumeration.md` |
| Listing (server-side) | `scm/<upstream>/repository-listing/` |
| Enrichment | `scm/<upstream>/repository-enrichment/` |
| Client-side | `scm/<upstream>/repository-enumeration/` |

| `upstream` | Listing | Enrichment | Enumeration |
| ---------- | ------- | ---------- | ----------- |
| `github` | [index][listing-github] | [index][enrichment-github] | [index][enumeration-github] |
| `gitlab` | [index][listing-gitlab] | [index][enrichment-gitlab] | [index][enumeration-gitlab] |

## Алгоритм (режим `--q*`, не `--scm-search`) {#алгоритм-q-star}

Нормативный порядок фаз:

1. **Листинг** — **всегда сначала** полностью завершить server-side перечисление

   по [base-enumeration][base-enumeration-github] и
   [repository-listing][listing-github] текущего `upstream`
   (параметры CLI → query, пагинация, [scope][hub-scope]
   и extension-опции на HTTP).
2. **Обогащение** — **затем** для **всех** карточек, полученных на шаге 1

   (не «по мере» листинга). Активные задания — реестр
   `repository-enrichment/` текущего `upstream` после резолва.
3. **Клиентская фильтрация** — **после** обогащения: сборка фильтров из

   Invocation (параметры `--qp`, `--qn`, `--qt`, `--qa` и extension, где
   применимо), затем [repository-enumeration][enumeration-github].
   **RetryingQueue на этом этапе не используется.**

### Подготовка клиентов

До HTTP: для текущего `upstream` выполнить шаги сборки из
[таблицы ArchiMate][archimate-scm]. Имена шагов процесса совпадают с
колонкой «Шаг в процессе». Тела HTTP-запросов — только в
`repository-listing/` и `repository-enrichment/`.

### Листинг и очередь

- Определить число страниц (или эквивалент пагинации API) и поставить в
  [RetryingQueue][queue] задание загрузки и разбора **каждой страницы**.
- Запустить очередь. Успешное задание дополняет массив карточек
  (RepositoryCards) по [base-enumeration][base-enumeration-github].
- Повторы объекта — [спека очереди][queue].
- **Прогресс-бар** двигается **по страницам** (один шаг — одна страница
  перечисления).

### Обогащение и очередь

- После **завершения** листинга множество работ — декартово произведение
  всех карточек набора и активных заданий реестра обогащения.
- Каждая пара (карточка, задание) — отдельный элемент [очереди][queue].
- Очередь вызывает у карточки приём задания. Карточка передаёт заданию порт
  записи. Задание само запрашивает SCM и через порт полностью заменяет поля,
  за которые отвечает. Шаг «Выполнить обогащение карточек» карточку не пишет.
- Повтор пары законен: замена полей задания полная. Граница повтора — целое
  задание, не отдельный HTTP-запрос. Правила повтора — [спека очереди][queue].
- **Прогресс-бар** двигается **на каждую** пару карточка + задание.
- Тела HTTP — файлы `repository-enrichment/<parameter>.md`.

### Клиентская фильтрация

- Фильтры строятся из Invocation после обогащения; применение — синхронный
  проход по карточкам ([enumeration][enumeration-github]).
- **RetryingQueue не используется**; сетевых запросов на этом этапе нет.

## Ветка `--scm-search`

Отдельный контур hub: один запрос листинга, **без** переносимых `--q*` и без
продуктового `scope` на запрос. Полный алгоритм выше относится к режиму `--q*`.

| `upstream` | Контракт argv и HTTP |
| ---------- | -------------------- |
| `github` | [repository-listing/scm-search.md][scm-search-github] |
| `gitlab` | [repository-listing/scm-search.md][scm-search-gitlab] |

## Не здесь

- Синтаксис и композиция `--q*`, ошибки до старта — hub.
- Политика `affiliated` / `all` — hub и [ADR-26102002][adr-saas].
- Контракт повтора объекта и квота HTTP — [очередь][queue] и
  [транспорт][transport].

<!-- LINKS -->
[card]: ../platform/scm-repository-card.md
[hub]: ../platform/scm-repository-selection.md
[hub-scope]: ../platform/scm-repository-selection.md#scope
[enumeration-github]: github/repository-enumeration/index.md
[enumeration-gitlab]: gitlab/repository-enumeration/index.md
[listing-github]: github/repository-listing/index.md
[listing-gitlab]: gitlab/repository-listing/index.md
[enrichment-github]: github/repository-enrichment/index.md
[enrichment-gitlab]: gitlab/repository-enrichment/index.md
[base-enumeration-github]: github/base-enumeration.md
[scm-search-github]: github/repository-listing/scm-search.md
[scm-search-gitlab]: gitlab/repository-listing/scm-search.md
[repository-selection]: ../../product-intent/capability/platform/repository-selection.md
[arch-readme]: ../../architecture/README.md
[archimate]: ../../architecture/git-herder.archimate
[archimate-scm]: #archimate-scm-application
[adr-hub]: ../../../ADR/ADR-26102201-scm-selection-hub-extension-specs.md
[adr-layout]: ../../../ADR/ADR-26110801-go-package-layout.md
[adr-saas]: ../../../ADR/ADR-26102002-public-saas-scm-affiliation-guard.md
[queue]: ../platform/retrying-queue.md
[transport]: ../platform/scm-http-transport.md
