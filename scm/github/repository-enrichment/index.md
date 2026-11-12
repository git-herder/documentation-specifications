# Реестр обогащения карточек (GitHub)

HTTP-дозагрузки полей [карточки][scm-repository-card] после листинга.
Оркестрация — [scm/index.md][scm-index]; листинг —
[repository-listing/][listing-index].

Каждая строка реестра — одно задание обогащения: когда оно активно и какой
HTTP выполняет, написано в файле строки. Пара «карточка × задание» как элемент
очереди и повтор этой пары — [очередь повторов][queue].

## Реестр заданий

| Поле | Файл | Условие после резолва |
| ---- | ---- | --------------------- |
| `properties` | [qa.md](qa.md) | задан хотя бы один `--qa` |

Для `--qt` отдельного HTTP обогащения **нет** — `labels` из ответа
Search ([listing/qt.md][listing-qt],
[base-enumeration § карточка][base-enumeration-card]).

<!-- LINKS -->
[scm-repository-card]: ../../../platform/scm-repository-card.md
[base-enumeration-card]: ../base-enumeration.md#карточка-репозитория
[scm-index]: ../../index.md
[queue]: ../../../platform/retrying-queue.md
[listing-index]: ../repository-listing/index.md
[listing-qt]: ../repository-listing/qt.md
