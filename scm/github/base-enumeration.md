# Базовое перечисление (GitHub)

SSOT для GitHub-адаптера: базовый поток получения списка репозиториев от SCM,
пагинация и **схема нормализованной карточки** (только GitHub; cross-SCM
обобщения нет).

Критерии CLI → REST — [repository-selection-mapping/](repository-selection-mapping/index.md).
Клиентская фильтрация по карточке — [repository-enumeration/](repository-enumeration/index.md).

## Базовый list

_(Эндпоинт(ы) первичного перечисления репозиториев — по мере фиксации контракта API.
Query от `scope` и других модификаторов — в mapping, не здесь.)_

## Пагинация

_(Правила обхода страниц для базового list — по мере фиксации.)_

## Карточка репозитория

Нормализованная запись одного репозитория после ответа SCM (имена полей SSOT здесь):

| Поле карточки | Назначение |
| ------------- | ---------- |
| `path` | Полный путь репозитория на платформе (owner/name) — для [enumeration/qp.md](repository-enumeration/qp.md) |
| `name` | Имя репозитория без пути — для [enumeration/qn.md](repository-enumeration/qn.md) |
| `topics` | Топики (если есть в API) — участвуют в [enumeration/qt.md](repository-enumeration/qt.md) |
| `labels` | Метки, если отличимы от topics — участвуют в qt |
| `custom_properties` | Пользовательские атрибуты — для [enumeration/qa.md](repository-enumeration/qa.md) |

Дозаполнение полей для `qt` / `qa` — отдельные HTTP-шаги в mapping; в карточку
пишутся уже нормализованные значения.

## Query `involves` (scope) {#query-involves-scope}

Имена REST query/path для продуктовой семантики **involves** при `scope=affiliated`
— SSOT здесь (детали поведения — [mapping/scope.md](repository-selection-mapping/scope.md)).

_(Конкретные имена параметров — по мере заполнения.)_

## Не описывать здесь

- Контракт CLI — [hub SCM-отбора][scm-repository-selection].
- Алгоритмы фильтрации `--q*` — [repository-enumeration/](repository-enumeration/index.md).

<!-- LINKS -->
[scm-repository-selection]: ../../platform/scm-repository-selection.md
