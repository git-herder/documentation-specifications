# Маппинг `qt` (GitLab)

CLI → HTTP. Смысл флагов — [hub § qt][scm-repository-selection-qt]. Отбор по
`=null` на клиенте — [enumeration/qt.md](../repository-enumeration/qt.md).

## Когда включать

Есть хотя бы одно условие `qt` после резолва argv и профиля. Если `qt` нет — этот
маппинг не делаем; `labels` как в [base-enumeration § карточка](../base-enumeration.md#карточка-репозитория).

## List проектов

`GET /api/v4/projects` — [base-enumeration](../base-enumeration.md). На каждую
страницу list вместе с [scope](scope.md) и [include-inactive](include-inactive.md):

| После резолва | GET-параметр |
| ------------- | ------------ |
| Нет условий **наличия** (`--qt` без `=null`, см. hub) | `topic` не добавлять |
| Есть одно или несколько **наличия** | `topic=<значение>` |

`<значение>` — имена тегов из условий наличия (разбор argv — hub § qt), через
запятую, **без пробелов**; каждое имя — urlencode для query. Порядок имён не важен.
Как GitLab трактует несколько topic в одном параметре — решает GitLab; мы только
передаём строку.

Условия `--qt …=null` на list **не** влияют.

## Дозагрузка `labels`

После того как собран весь list (все страницы), **для каждого** проекта:

| | |
| --- | --- |
| Запрос | `GET /api/v4/projects/:id/labels`, `:id` = `uid` карточки |
| Результат | Перезаписать `labels` карточки |

Из ответа (массив JSON) взять у каждого элемента поле `name`, оставить **уникальные**
строки — это новый `labels`. Topics из list **не** добавлять. Нет имён — `[]`.

Progress bar на этом проходе здесь не описываем (см. list в base-enumeration).

## Ошибка `GET …/labels`

Сначала повторы по [`--retries-number`][cli-retries], потом [`--on-error`][cli-on-error]:

| `--on-error` | Что делать |
| ------------ | ---------- |
| `abort` | Остановить команду |
| `continue` | Для этого проекта не менять `labels` (остаются topics после list); идти дальше |

Подробнее про повторы — [cli/index.md][cli-retries].

## Порядок шагов

1. List (`scope`, `include-inactive`, при необходимости `topic`).
2. Дозагрузка labels, если после резолва есть `qt`.
3. Локальные фильтры — [enumeration/](../repository-enumeration/index.md).

## Не здесь

- Разбор `--qt` и AND между флагами — hub.
- Фильтр `=null` — [enumeration/qt.md](../repository-enumeration/qt.md).

<!-- LINKS -->
[scm-repository-selection-qt]: ../../../platform/scm-repository-selection.md#qt-теги
[cli-retries]: ../../../cli/index.md#повторы-и---on-error
[cli-on-error]: ../../../cli/index.md#--on-error
