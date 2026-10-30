# Клиентская фильтрация `qt` (GitHub)

## Контракт

[hub § qt][scm-repository-selection-qt]. Поле `labels` карточки —
[scm-repository-card][scm-repository-card] (из `topics` Search list).

На list **нет** server-side фильтра по тегам ([mapping/qt.md](../repository-selection-mapping/qt.md)).

## Вход

Карточки после полного list (все страницы) и постраничного enrich `--qa`, если он
был.

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
