# Обобщённая карточка репозитория (SCM)

Cross-SCM **нормализованная запись** одного репозитория после ответа [SCM API][SCM-PLATFORM-TERM]
и обогащения адаптера. Единственный SSOT имён полей и типов карточки для всех
`upstream`.

Замысел: [единый отбор для массовых операций][repository-selection-capability].
Перечисление и локальная фильтрация: [ADR-26102001][adr-local-eval],
[SCM-отбор (hub)][scm-repository-selection]. Раскладка hub / base-enumeration /
listing / enrichment / enumeration: [ADR-26102201][adr-hub-extension].

## Трассировка вверх

- [Единый отбор для массовых операций][repository-selection-capability]
- [ADR-26102001: локальная оценка критериев SCM-отбора][adr-local-eval]
- [ADR-26102201: hub и extension-спеки SCM-отбора][adr-hub-extension]

## Назначение

SCM-адаптер **наполняет** карточку для каждого репозитория в выборке. Карточка —
вход для клиентской фильтрации `--qp`, `--qn`, `--qt`, `--qa` и для JSON-вывода
команд, которые явно ссылаются на эту спеку (например [report](../cli/report/index.md)).

Правила native API → поля карточки из list — **только** в
`scm/<upstream>/base-enumeration.md` (дозагрузки — `repository-enrichment/`).
Здесь — **только** обобщённая форма.

## Поля

| Поле | Тип | Инварианты |
| ---- | --- | ---------- |
| `uid` | string **или** number (JSON) | Уникальный идентификатор репозитория **в пределах одного** подключения к SCM (резолв `upstream`, `host` и учётных данных профиля). **Не** используется в критериях `--q*`. JSON-тип и правило построения значения зависят от `upstream` — [ниже](#uid). |
| `name` | UTF-8 string | Имя репозитория **без** префикса пути; пример `specifications`. |
| `path` | UTF-8 string | Путь от **корня SCM-контекста** (иерархия namespace на платформе) до **родителя** `name`; сегменты через `/`. Пример: `snowbridge-projects/git-herder/documentation`. **`name` не повторяют** в конце `path`. |
| `url` | UTF-8 string | Абсолютный URL страницы репозитория в UI SCM (схема `http` или `https`), непустая строка. **Не** используется в критериях `--q*`. Заполняется из list, не заданиями обогащения. Native-поле — только `base-enumeration` per `upstream`. |
| `description` | UTF-8 string | Неограниченная длина. Если у SCM нет описания — **`""`**; поле **всегда** присутствует. |
| `labels` | массив string | Каждый элемент — один **тег** (или native-аналог, сведённый к строке тега) в UTF-8. Пустой массив `[]`, если тегов нет. Порядок **не** нормативен; повторы допустимы. Маппинг native → строки — `base-enumeration` per `upstream`. |
| `properties` | объект string → scalar | Ключ — UTF-8 string (имя property). Значение — ровно один scalar: string, number (JSON number), boolean или **date** (JSON string). **Date** в карточке — профиль **data-timestamp** ([ADR-26101704][adr-timestamp]). Пустой объект `{}`, если custom properties нет. |

## JSON

Нормативная сериализация — один JSON-объект с **ровно** семью ключами верхнего
уровня: `uid`, `name`, `path`, `url`, `description`, `labels`, `properties`.
Типы — как в таблице.

Пример (GitHub-подобный `uid` — строка):

```json
{
  "uid": "snowbridge-projects/git-herder/documentation/specifications",
  "name": "specifications",
  "path": "snowbridge-projects/git-herder/documentation",
  "url": "https://scm.example/snowbridge-projects/git-herder/documentation/specifications",
  "description": "Прикладные спецификации Git Herder",
  "labels": [
    "foo",
    "bar",
    "documentation"
  ],
  "properties": {
    "tier": "gold",
    "reviewers": 3,
    "archived": false,
    "review_by": "2026-12-31T23:59:59.000+00:00"
  }
}
```

## Связь с критериями SCM-отбора

Операторы и композиция условий — [hub][scm-repository-selection]; здесь — **какие
поля карточки** участвуют.

| Критерий | Поле карточки |
| -------- | ------------- |
| `--qp` | `path` |
| `--qn` | `name` |
| `--qt` | `labels` |
| `--qa` | `properties` |

### `qt` и `labels`

Критерий `--qt` — [hub § qt][scm-repository-selection-qt] (строгое наличие / отсутствие
тега; разбор argv только в hub). Поле карточки — массив `labels`. Алгоритм
сопоставления после разбора условий — `scm/<upstream>/repository-enumeration/qt.md`
(без дублирования hub).

### `qa` и `properties`

Условия `--qa` оценивают значения в `properties` по правилам hub
([§ qa][scm-repository-selection-qa]). Для операторов `<`, `>`, `<=`, `>=` и для
равенства сравнивают **каноническое строковое представление** scalar (таблица ниже),
лексикографически.

| Тип в `properties` | Каноническая строка для сравнения |
| ------------------ | --------------------------------- |
| string | значение как есть (UTF-8) |
| number | десятичная запись без экспоненты; целые без `.0`; отрицательные с `-` |
| boolean | `true` или `false` (нижний регистр) |
| date (string) | строка **data-timestamp** после нормализации адаптером |

Per-SCM алгоритм — `scm/<upstream>/repository-enumeration/qa.md`.

### `uid`

Стабильный идентификатор для дедупликации, журнала и внутренних ссылок в процессе
Git Herder. Два репозитория с разными `uid` **не** считаются одним объектом; совпадение
`uid` в одной выборке — одна и та же сущность на SCM.

| `upstream` | JSON-тип `uid` | SSOT формы значения |
| ---------- | -------------- | ------------------- |
| `github` | string | [base-enumeration (GitHub)](../scm/github/base-enumeration.md#uid) |
| `gitlab` | number | [base-enumeration (GitLab)](../scm/gitlab/base-enumeration.md#uid) |
| иные | по спеке upstream | `scm/<upstream>/base-enumeration.md#uid` |

Форма `uid` для каждого `upstream` задаётся **только** в `base-enumeration`
(GitHub — строка из `path` и `name`; GitLab — id проекта из API).

### `url`

Абсолютный URL страницы репозитория в UI SCM. Какое native-поле копировать в
`url` — только `base-enumeration` выбранного `upstream` (ветка `--scm-search`
наполняет карточку по той же таблице).

Если в элементе list значение отсутствует, равно `null` или пустое после trim —
элемент **невалиден**. Задание страницы list завершается ошибкой; повторы и
итог — [`--retries-number` / `--on-error`][cli-on-error].

## Инварианты

- Все семь ключей верхнего уровня **обязательны** в каждой карточке.
- `url` непустой после list; задания обогащения его **не** заменяют.
- `labels` — массив (не `null`); минимум `[]`. `properties` — объект (не `null`);
  минимум `{}`.
- Семантика полей карточки **не** дублируется в hub и в
  `repository-enumeration/*` — только ссылки на эту спеку.

## Не описывать здесь

- Базовый list, листинг, обогащение — `scm/<upstream>/base-enumeration.md`,
  `repository-listing/`, `repository-enrichment/`; оркестрация — [scm/index.md](../scm/index.md).
- Таблицы переносимых и extension-опций — [scm-repository-selection.md](scm-repository-selection.md).
- Реализацию в `application/`.

<!-- LINKS -->
[SCM-PLATFORM-TERM]: ../../../glossary/terms/scm-platform.md
[repository-selection-capability]: ../../product-intent/capability/platform/repository-selection.md
[adr-local-eval]: ../../../ADR/ADR-26102001-scm-repository-selection-evaluation.md
[adr-hub-extension]: ../../../ADR/ADR-26102201-scm-selection-hub-extension-specs.md
[adr-timestamp]: ../../../ADR/ADR-26101704-timestamp-formats.md
[scm-repository-selection]: scm-repository-selection.md
[scm-repository-selection-qt]: scm-repository-selection.md#qt-теги
[scm-repository-selection-qa]: scm-repository-selection.md#qa-атрибуты
[cli-on-error]: ../cli/index.md#повторы-и---on-error
