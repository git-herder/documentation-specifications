# Обогащение `labels` (GitLab)

Дозагрузка поля `labels` после листинга. Смысл `--qt` — [hub § qt][scm-repository-selection-qt];
server-side `topic` на list — [listing/qt.md](../repository-listing/qt.md).

## Когда включать

После резолва задан хотя бы один `--qt`. Иначе этот джоб **не** ставим в очередь.

## HTTP

Для **каждой** карточки в наборе после листинга:

| | |
| --- | --- |
| Запрос | `GET /api/v4/projects/:id/labels`, `:id` = `uid` карточки |
| Результат | Перезаписать `labels` карточки |

Из ответа (массив JSON) взять у каждого элемента поле `name`, оставить **уникальные**
строки — это новый `labels`. Topics из list **не** добавлять. Нет имён — `[]`.

## Ошибка `GET …/labels`

Сначала повторы по [`--retries-number`][cli-retries], потом [`--on-error`][cli-on-error]:

| `--on-error` | Что делать |
| ------------ | ---------- |
| `abort` | Остановить команду |
| `continue` | Для этого проекта не менять `labels` (остаётся наполнение из `topics` по base-enumeration); идти дальше |

Подробнее про повторы — [cli/index.md][cli-retries].

## Не здесь

- Разбор `--qt` и AND между флагами — hub.
- Фильтр `=null` — [enumeration/qt.md](../repository-enumeration/qt.md).
- Порядок фаз и прогресс очереди — [scm/index.md](../../index.md).

<!-- LINKS -->
[scm-repository-selection-qt]: ../../../platform/scm-repository-selection.md#qt-теги
[cli-retries]: ../../../cli/index.md#повторы-и---on-error
[cli-on-error]: ../../../cli/index.md#--on-error
