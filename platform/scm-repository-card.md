# Обобщённая карточка репозитория (SCM)

Cross-SCM **нормализованная запись** одного репозитория после ответа [SCM API][SCM-PLATFORM-TERM]
и обогащения адаптера. Единственный SSOT имён полей и типов карточки для всех
`upstream`.

Замысел: [единый отбор для массовых операций][repository-selection-capability].
Перечисление и локальная фильтрация: [ADR-26102001][adr-local-eval],
[SCM-отбор (hub)][scm-repository-selection]. Раскладка hub / base-enumeration /
mapping / enumeration: [ADR-26102201][adr-hub-extension].

## Трассировка вверх

- [Единый отбор для массовых операций][repository-selection-capability]
- [ADR-26102001: локальная оценка критериев SCM-отбора][adr-local-eval]
- [ADR-26102201: hub и extension-спеки SCM-отбора][adr-hub-extension]

## Назначение

SCM-адаптер **наполняет** карточку для каждого репозитория в выборке. Карточка —
вход для клиентской фильтрации `--qp`, `--qn`, `--qt`, `--qa` и для JSON-вывода
команд, которые явно ссылаются на эту спеку (например [report](../cli/report/index.md)).

Правила маппинга native API → поля карточки — **только** в
`scm/<upstream>/base-enumeration.md` (и REST-дозагрузка — mapping). Здесь — **только**
обобщённая форма.

## Поля

| Поле | Тип | Инварианты |
| ---- | --- | ---------- |
| `uid` | string **или** number (JSON) | Уникальный идентификатор репозитория **в пределах одного** подключения к SCM (резолв `upstream`, `host` и учётных данных профиля). **Не** используется в критериях `--q*`. JSON-тип и правило построения значения зависят от `upstream` — [ниже](#uid). |
| `name` | UTF-8 string | Имя репозитория **без** префикса пути; пример `specifications`. |
| `path` | UTF-8 string | Путь от **корня SCM-контекста** (иерархия namespace на платформе) до **родителя** `name`; сегменты через `/`. Пример: `snowbridge-projects/git-herder/documentation`. **`name` не повторяют** в конце `path`. |
| `description` | UTF-8 string | Неограниченная длина. Если у SCM нет описания — **`""`**; поле **всегда** присутствует. |
| `labels` | массив string | Каждый элемент — одна метка (тег, топик, лейбл или аналог на SCM) в нормализованном UTF-8 виде. Пустой массив `[]`, если меток нет. Порядок **не** нормативен; повторы допустимы. Как native API кладётся в строки — `base-enumeration` per `upstream`. |
| `properties` | объект string → scalar | Ключ — UTF-8 string (имя property). Значение — ровно один scalar: string, number (JSON number), boolean или **date** (JSON string). **Date** в карточке — профиль **data-timestamp** ([ADR-26101704][adr-timestamp]). Пустой объект `{}`, если custom properties нет. |

## JSON

Нормативная сериализация — один JSON-объект с **ровно** шестью ключами верхнего
уровня: `uid`, `name`, `path`, `description`, `labels`, `properties`. Типы — как в таблице.

Пример (GitHub-подобный `uid` — строка):

```json
{
  "uid": "snowbridge-projects/git-herder/documentation/specifications",
  "name": "specifications",
  "path": "snowbridge-projects/git-herder/documentation",
  "description": "Прикладные спецификации Git Herder",
  "labels": [
    "team=Alpha",
    "env=prod",
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

Условие `--qt` ([hub § qt][scm-repository-selection-qt]) проверяют по **элементам**
массива `labels`. Адаптер добавляет в массив строки для всех native-источников класса
«теги/топики/лейблы»; продуктовый OR на одной SCM — за счёт **нескольких** элементов
или форм строк (маппинг — `base-enumeration`).

Сопоставление строки условия `key=value` / `key!=value` с элементами — per-SCM
`repository-enumeration/qt.md` (без дублирования таблицы операторов hub).

Per-SCM алгоритм проверки — `scm/<upstream>/repository-enumeration/qt.md`.

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

Форма `uid` для каждого `upstream` задаётся **только** в `base-enumeration` (GitHub —
строка из `path` и `name`; GitLab — id проекта из API).

## Инварианты

- Все шесть ключей верхнего уровня **обязательны** в каждой карточке.
- `labels` — массив (не `null`); минимум `[]`. `properties` — объект (не `null`); минимум `{}`.
- Семантика полей карточки **не** дублируется в hub и в `repository-enumeration/*` —
  только ссылки на эту спеку.

## Не описывать здесь

- Базовый list, пагинация, REST — `scm/<upstream>/base-enumeration.md`, mapping.
- Таблицы переносимых и extension-опций — [scm-repository-selection.md](scm-repository-selection.md).
- Реализацию в `application/`.

<!-- LINKS -->
[SCM-PLATFORM-TERM]: ../../../glossary/terms/scm-platform.md
[repository-selection-capability]: ../../product-intent/capability/platform/repository-selection.md
[adr-local-eval]: ../../../ADR/ADR-26102001-scm-repository-selection-evaluation.md
[adr-hub-extension]: ../../../ADR/ADR-26102201-scm-selection-hub-extension-specs.md
[adr-timestamp]: ../../../ADR/ADR-26101704-timestamp-formats.md
[scm-repository-selection]: scm-repository-selection.md
[scm-repository-selection-qt]: scm-repository-selection.md#qt-теги-топики-лейблы
[scm-repository-selection-qa]: scm-repository-selection.md#qa-атрибуты
