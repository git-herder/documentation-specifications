# Клиентская фильтрация SCM-отбора (GitLab)

[base-enumeration.md](../base-enumeration.md), [hub][scm-repository-selection],
[ADR-26102001][adr-local-eval].

## Реестр

| Параметр | Hub | Файл | Поля |
| -------- | --- | ---- | ---- |
| `qp` | [§ qp/qn][scm-repository-selection-qp-qn] | [qp.md](qp.md) | `path` |
| `qn` | [§ qp/qn][scm-repository-selection-qp-qn] | [qn.md](qn.md) | `name` |
| `qt` | [§ qt][scm-repository-selection-qt] | [qt.md](qt.md) | `labels` |
| `qa` | [§ qa][scm-repository-selection-qa] | [qa.md](qa.md) | custom_properties |

`scope`, `include-inactive`, `scm-search` — только mapping.

## Порядок

Локальные фильтры после list + mapping enrich; AND по hub.

<!-- LINKS -->
[scm-repository-selection]: ../../../platform/scm-repository-selection.md
[scm-repository-selection-qp-qn]: ../../../platform/scm-repository-selection.md#qp-и-qn
[scm-repository-selection-qt]: ../../../platform/scm-repository-selection.md#qt-теги
[scm-repository-selection-qa]: ../../../platform/scm-repository-selection.md#qa-атрибуты
[adr-local-eval]: ../../../../../ADR/ADR-26102001-scm-repository-selection-evaluation.md
