# Реестр обогащения карточек (GitHub)

HTTP-дозагрузки полей [карточки][scm-repository-card] после листинга.
Оркестрация — [scm/index.md](../../index.md); листинг —
[repository-listing/](../repository-listing/index.md).

Контракт единицы «джоб обогащения» в очереди — отдельная будущая спека.

## Реестр джобов

| Поле | Файл | Условие после резолва |
| ---- | ---- | --------------------- |
| `properties` | [qa.md](qa.md) | задан хотя бы один `--qa` |

Для `--qt` отдельного HTTP обогащения **нет** — `labels` из ответа Search
([listing/qt.md](../repository-listing/qt.md), [base-enumeration § карточка][base-enumeration-card]).

<!-- LINKS -->
[scm-repository-card]: ../../../platform/scm-repository-card.md
[base-enumeration-card]: ../base-enumeration.md#карточка-репозитория
