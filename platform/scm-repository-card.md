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
| `name` | UTF-8 string | Имя репозитория **без** префикса пути; пример `specifications`. |
| `path` | UTF-8 string | Путь от **корня SCM-контекста** (иерархия namespace на платформе) до **родителя** `name`; сегменты через `/`. Пример: `snowbridge-projects/git-herder/documentation`. **`name` не повторяют** в конце `path`. |
| `description` | UTF-8 string | Неограниченная длина. Если у SCM нет описания — **`""`**; поле **всегда** присутствует. |
| `labels` | объект string → string | Ключ и значение — неограниченные UTF-8 **строки**; значение **не** number, boolean или date. Пустой объект `{}`, если меток нет. Один JSON-ключ — одно значение; коллизии при слиянии нескольких native-источников — в `base-enumeration` per `upstream`. |
| `properties` | объект string → scalar | Ключ — UTF-8 string (имя property). Значение — ровно один scalar: string, number (JSON number), boolean или **date** (JSON string). **Date** в карточке — профиль **data-timestamp** ([ADR-26101704][adr-timestamp]). Пустой объект `{}`, если custom properties нет. |

## JSON

Нормативная сериализация — один JSON-объект с **ровно** пятью ключами верхнего
уровня: `name`, `path`, `description`, `labels`, `properties`. Типы — как в таблице.

Пример:

```json
{
  "name": "specifications",
  "path": "snowbridge-projects/git-herder/documentation",
  "description": "Прикладные спецификации Git Herder",
  "labels": {
    "team": "Alpha",
    "env": "prod"
  },
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

Условие `--qt key=value` (или `key!=value`) смотрит на map `labels`: ключ **должен
присутствовать** в объекте; значение сравнивается с `labels[key]` как со **строкой**
(см. hub). Продуктовый OR между несколькими native-реализациями класса «теги/топики/лейблы»
на одной SCM ([hub § qt][scm-repository-selection-qt]) достигается тем, как адаптер
**заполняет** `labels` до фильтрации (per-SCM правила — `base-enumeration`).

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

## Инварианты

- Все пять ключей верхнего уровня **обязательны** в каждой карточке.
- `labels` и `properties` — объекты (не `null`); минимум `{}`.
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
