# Загрузка из SCM и обогащение карточек репозиториев

SSOT **оркестрации** процесса ArchiMate «Загрузка из SCM и обогащение карточек
репозиториев» (`id-649305ae`). Переиспользуется командами,
которым нужен набор [карточек репозитория](../platform/scm-repository-card.md) из
SCM.

Критерии отбора, резолв argv и профиля —
[hub SCM-отбора](../platform/scm-repository-selection.md).
HTTP-контракты листинга и обогащения — per `upstream` ниже; клиентская
фильтрация — [repository-enumeration/](github/repository-enumeration/index.md).

## Трассировка вверх

- [Единый отбор для массовых операций](../../product-intent/capability/platform/repository-selection.md)
- Навигация по модели: [architecture/README.md](../../architecture/README.md);
  артефакт [git-herder.archimate](../../architecture/git-herder.archimate)
- [ADR-26102201](../../../ADR/ADR-26102201-scm-selection-hub-extension-specs.md)
  (раскладка hub / listing / enrichment / enumeration)
- Компоновка кода: [ADR-26110801](../../../ADR/ADR-26110801-go-package-layout.md)

## Трассировка ArchiMate (приложение SCM) {#archimate-scm-application}

View **SCM Listing & Enrichment** и процесс `id-649305ae` (см. **Report command
flow** в модели) описывают один контур: сначала **сборка** клиентов, затем
**runtime** HTTP через готовые интерфейсы.

| Фаза | ArchiMate (интерфейсы / сервисы) | Шаг в процессе `id-649305ae` |
| ---- | -------------------------------- | ---------------------------- |
| Сборка листинга | ScmClientFactory, ScmListingBuilder, сервис «Создание listing-клиента» | «Собрать REST-клиент server-side отборов для текущего upstream» |
| Сборка обогащения | ScmClientFactory, ScmEnrichmentBuilder, сервис «Создание enrich-клиентов» | «Собрать REST-клиент получения дополнительных данных для обогащения карточки» |
| Листинг (runtime) | **ScmListing** (не Builder) | «Выполнить парсинг страниц…» |
| Обогащение (runtime) | **ScmEnrich** | «Выполнить обогащение карточек» |

Builder'ы — только конфигурация запроса перед `Build()`; пагинация и дозагрузки
в пайплайне вызывают **ScmListing** / **ScmEnrich**. Детали view —
[architecture/README.md][arch-readme].

## Слои per upstream

| Слой | Путь |
| ---- | ---- |
| Hub | [platform/scm-repository-selection.md](../platform/scm-repository-selection.md) |
| Options | `scm/<upstream>/repository-selection-options.md` |
| Base list / карточка из list | `scm/<upstream>/base-enumeration.md` |
| Listing (server-side) | `scm/<upstream>/repository-listing/` |
| Enrichment | `scm/<upstream>/repository-enrichment/` |
| Client-side | `scm/<upstream>/repository-enumeration/` |

| `upstream` | Listing | Enrichment | Enumeration |
| ---------- | ------- | ---------- | ----------- |
| `github` | [index](github/repository-listing/index.md) | [index](github/repository-enrichment/index.md) | [index](github/repository-enumeration/index.md) |
| `gitlab` | [index](gitlab/repository-listing/index.md) | [index](gitlab/repository-enrichment/index.md) | [index](gitlab/repository-enumeration/index.md) |

## Алгоритм (режим `--q*`, не `--scm-search`) {#алгоритм-q-star}

Нормативный порядок фаз:

1. **Листинг** — **всегда сначала** полностью завершить server-side перечисление

   по [base-enumeration](github/base-enumeration.md) и
    [repository-listing](github/repository-listing/index.md) текущего `upstream`
    (параметры CLI → query, пагинация, [scope](../platform/scm-repository-selection.md#scope)
    и extension-опции на HTTP).
2. **Обогащение** — **затем** для **всех** карточек, полученных на шаге 1

   (не «по мере» листинга). Активные джобы — реестр
    [repository-enrichment/index.md](gitlab/repository-enrichment/index.md) per
    `upstream` после резолва.
3. **Клиентская фильтрация** — **после** обогащения: сборка фильтров из

   Invocation (параметры `--qp`, `--qn`, `--qt`, `--qa` и extension, где
    применимо), затем [repository-enumeration](github/repository-enumeration/index.md).
    **RetryingQueue на этом этапе не используется.**

### Подготовка REST-клиентов

До HTTP: для текущего `upstream` выполнить шаги сборки из
[таблицы ArchiMate][archimate-scm] (ScmClientFactory → Builder → клиент
**ScmListing** / **ScmEnrich**). Имена process-функций в модели совпадают с
колонкой «Шаг в процессе» в той же таблице. Прикладной контракт клиентов —
интерфейсы в ArchiMate; тела HTTP-запросов — только в
`repository-listing/` и `repository-enrichment/`.

### Листинг и RetryingQueue

- Определить число страниц (или эквивалент пагинации API) и поставить в
**RetryingQueue** джобы загрузки и разбора **каждой страницы**.
- Запустить очередь; каждый успешный джоб дополняет массив карточек
(RepositoryCards) по [base-enumeration](github/base-enumeration.md).
- **Прогресс-бар** двигается **по страницам** (один шаг — одна страница list).

### Обогащение и RetryingQueue

- После **завершения** листинга: множество работ = **декартово произведение**
{все карточки набора} × {все активные джобы из реестра enrichment}.
- Каждая пара (карточка, джоб) — отдельный элемент **RetryingQueue**.
- **Прогресс-бар** двигается **на каждую** пару карточка + джоб.
- Успешный джоб обновляет соответствующую карточку in-place (HTTP — файлы
`repository-enrichment/<parameter>.md`).
- Семантика абстракции «джоб обогащения» как единицы API очереди — **отдельная
будущая спека**; здесь зафиксирован только порядок фазы и связь с реестром.

### Клиентская фильтрация

- Фильтры строятся из Invocation после обогащения; применение — синхронный проход
по карточкам ([enumeration](github/repository-enumeration/index.md)).
- **RetryingQueue не используется**; сетевых запросов на этом этапе нет.

## Ветка `--scm-search`

Отдельный контур hub: один запрос листинга по
[repository-listing/scm-search.md](gitlab/repository-listing/scm-search.md) (GitLab;
у GitHub — свой файл при появлении), **без** переносимых `--q`* и без продуктового
`scope` на запрос. Полный алгоритм выше относится к режиму `--q*`.

## Не здесь

- Синтаксис и композиция `--q*`, ошибки до старта — hub.
- Политика `affiliated` / `all` — hub и [ADR-26102002](../../../ADR/ADR-26102002-public-saas-scm-affiliation-guard.md).

<!-- LINKS -->
[arch-readme]: ../../architecture/README.md
[archimate-scm]: #archimate-scm-application
