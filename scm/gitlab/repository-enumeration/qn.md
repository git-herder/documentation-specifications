# Клиентская фильтрация `qn` (GitLab)

## Контракт

[hub § qp и qn][scm-repository-selection-qp-qn]. Проверяем `name` карточки
[scm-repository-card][scm-repository-card].

Запросов к API нет.

## Данные на карточке

Как в [qp.md](qp.md): **`name`** и связанные поля — см. § «Данные на карточке» там.

## Алгоритм

Для каждого `--qn` после резолва — те же шаги, что `--qp` в [qp.md](qp.md), но
строка для проверки — **`name`**.

Несколько `--qn` — **AND** с остальными критериями (hub).

<!-- LINKS -->
[scm-repository-selection-qp-qn]: ../../../platform/scm-repository-selection.md#qp-и-qn
[scm-repository-card]: ../../../platform/scm-repository-card.md
