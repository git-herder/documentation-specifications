# Клиентская фильтрация `qp` (GitLab)

## Контракт

[hub § qp и qn][scm-repository-selection-qp-qn].

## Вход

`path` — [base-enumeration.md](../base-enumeration.md).

## Алгоритм

Для каждого `--qp`: регэкп (re2) по `path` или подстрока; AND с остальными критериями.

<!-- LINKS -->
[scm-repository-selection-qp-qn]: ../../../platform/scm-repository-selection.md#qp-и-qn
