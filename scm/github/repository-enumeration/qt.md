# Клиентская фильтрация `qt` (GitHub)

## Контракт

[hub § qt][scm-repository-selection-qt]. Поле `labels` карточки —
[scm-repository-card][scm-repository-card] (из `topics` Search list).

На list **нет** server-side фильтра по тегам ([listing/qt.md](../repository-listing/qt.md)).

## Данные на карточке

Поле **`labels`**: из Search list ([base-enumeration § карточка](../base-enumeration.md#карточка-репозитория),
[listing/qt.md](../repository-listing/qt.md)).

При резолве **`qa`**: поле **`properties`** — [enrichment/qa.md](../repository-enrichment/qa.md);
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
