# Реестр маппинга SCM-отбора (GitLab)

CLI → HTTP. [base-enumeration.md](../base-enumeration.md),
[repository-enumeration/](../repository-enumeration/index.md),
[options](../repository-selection-options.md), [hub][scm-repository-selection].

## Реестр (REST)

| Параметр | Контракт | Маппинг |
| -------- | -------- | ------- |
| `scope` | [hub § scope][scm-repository-selection-scope] | [scope.md](scope.md) |
| `include-inactive` | [options § include-inactive][options-include-inactive] | [include-inactive.md](include-inactive.md) |
| `scm-search` | [hub § scm-search][scm-repository-selection-scm-search] | [scm-search.md](scm-search.md) |
| `qt` | [hub § qt][scm-repository-selection-qt] | [qt.md](qt.md) |
| `qa` | [hub § qa][scm-repository-selection-qa] | [qa.md](qa.md) |

`qp`, `qn` — только [enumeration/](../repository-enumeration/index.md).

<!-- LINKS -->
[scm-repository-selection]: ../../../platform/scm-repository-selection.md
[scm-repository-selection-scope]: ../../../platform/scm-repository-selection.md#scope
[scm-repository-selection-scm-search]: ../../../platform/scm-repository-selection.md#scm-search
[scm-repository-selection-qt]: ../../../platform/scm-repository-selection.md#qt-теги
[scm-repository-selection-qa]: ../../../platform/scm-repository-selection.md#qa-атрибуты
[options-include-inactive]: ../repository-selection-options.md#include-inactive
