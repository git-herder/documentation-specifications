# Листинг `qt` (GitLab)

CLI → HTTP (server-side list). Смысл флагов — [hub § qt][scm-repository-selection-qt].
Отбор по `=null` на клиенте — [enumeration/qt.md](../repository-enumeration/qt.md).
Дозагрузка `labels` — [enrichment/qt.md](../repository-enrichment/qt.md).

## Когда включать

Есть хотя бы одно условие `qt` после резолва argv и профиля. Если `qt` нет — этот
листинг не меняем; `labels` как в
[base-enumeration § карточка](../base-enumeration.md#карточка-репозитория).

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

## Не здесь

- Разбор `--qt` и AND между флагами — hub.
- Фильтр `=null` — [enumeration/qt.md](../repository-enumeration/qt.md).
- Дозагрузка `labels` — [enrichment/qt.md](../repository-enrichment/qt.md).

<!-- LINKS -->
[scm-repository-selection-qt]: ../../../platform/scm-repository-selection.md#qt-теги
