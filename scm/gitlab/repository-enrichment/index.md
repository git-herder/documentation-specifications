# Реестр обогащения карточек (GitLab)

HTTP-дозагрузки полей [карточки][scm-repository-card] после листинга.
Оркестрация (порядок фаз, прогресс) — [scm/index.md][scm-index];
листинг — [repository-listing/][listing-index].

Каждая строка реестра — одно задание обогащения. Здесь — когда оно активно и
что делает HTTP. Пара «карточка × задание» и её повтор — [очередь
повторов][queue].

## Реестр заданий

| Поле | Файл | Условие после резолва |
| ---- | ---- | --------------------- |
| `labels` | [qt.md](qt.md) | задан хотя бы один `--qt` |
| `properties` | [qa.md](qa.md) | задан хотя бы один `--qa` |

<!-- LINKS -->
[scm-repository-card]: ../../../platform/scm-repository-card.md
[scm-index]: ../../index.md
[queue]: ../../../platform/retrying-queue.md
[listing-index]: ../repository-listing/index.md
