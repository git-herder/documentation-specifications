# Клиентская фильтрация `qt` (GitHub)

## Контракт

[hub § qt][scm-repository-selection-qt]. Поле `labels` карточки —
[scm-repository-card][scm-repository-card] (из `topics` Search list).

На list **нет** server-side фильтра по тегам ([mapping/qt.md](../repository-selection-mapping/qt.md)).

## Данные на карточке

Поле **`labels`**: из Search list ([base-enumeration § карточка](../base-enumeration.md#карточка-репозитория),
[mapping/qt.md](../repository-selection-mapping/qt.md)).

При резолве **`qa`**: поле **`properties`** — по [mapping/qa.md](../repository-selection-mapping/qa.md);
иначе `{}` как в [base-enumeration § карточка](../base-enumeration.md#карточка-репозитория).

## Алгоритм

Для каждого `--qt` после резолва (разбор — hub § qt):

| Вид условия | На клиенте |
| ----------- | ---------- |
| Наличие (без `=null`) | Пропустить репозиторий, если *имя* **нет** в `labels`; иначе оставить |
| Отсутствие (`=null`) | Пропустить репозиторий, если *имя* **есть** в `labels`; иначе оставить |

Сравнение *имени* с элементами `labels` — **точное**, с учётом регистра.

Несколько `--qt` — **AND** (hub). То же AND с клиентской частью `--qa`.

<!-- LINKS -->
[scm-repository-selection-qt]: ../../../platform/scm-repository-selection.md#qt-теги
[scm-repository-card]: ../../../platform/scm-repository-card.md
