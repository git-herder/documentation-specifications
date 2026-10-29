# Клиентская фильтрация `qn` (GitHub)

## Контракт

[hub § qp и qn][scm-repository-selection-qp-qn].

## Вход

Поле **`name`** карточки ([base-enumeration.md](../base-enumeration.md)).

## Алгоритм

Для каждого `--qn`: регэкп или подстрока по `name`, как для `qp` по `path` в
[qp.md](qp.md). AND с остальными критериями.

<!-- LINKS -->
[scm-repository-selection-qp-qn]: ../../../platform/scm-repository-selection.md#qp-и-qn
