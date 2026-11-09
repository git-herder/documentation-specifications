# Реестр обогащения карточек (GitLab)

HTTP-дозагрузки полей [карточки][scm-repository-card] после листинга.
Оркестрация (очередь, прогресс) — [scm/index.md](../../index.md);
листинг — [repository-listing/](../repository-listing/index.md).

Контракт единицы «джоб обогащения» в очереди — отдельная будущая спека; здесь —
когда джоб активен и что делает HTTP.

## Реестр джобов

| Поле | Файл | Условие после резолва |
| ---- | ---- | --------------------- |
| `labels` | [qt.md](qt.md) | задан хотя бы один `--qt` |
| `properties` | [qa.md](qa.md) | задан хотя бы один `--qa` |

<!-- LINKS -->
[scm-repository-card]: ../../../platform/scm-repository-card.md
