# Клиентская фильтрация SCM-отбора (GitHub)

SSOT: как **уже полученная** от SCM выборка (карточки из [base-enumeration.md](../base-enumeration.md))
фильтруется в процессе Git Herder. REST — только в [mapping](../repository-selection-mapping/index.md).

Продуктовый контракт операторов и AND/OR — [hub][scm-repository-selection]. Принцип
локальной оценки — [ADR-26102001][adr-local-eval].

## Реестр параметров

| Параметр | Контракт (hub) | Файл | Поля карточки |
| -------- | -------------- | ---- | ------------- |
| `qp` | [§ qp/qn][scm-repository-selection-qp-qn] | [qp.md](qp.md) | `path` |
| `qn` | [§ qp/qn][scm-repository-selection-qp-qn] | [qn.md](qn.md) | `name` |
| `qt` | [§ qt][scm-repository-selection-qt] | [qt.md](qt.md) | `labels` |
| `qa` | [§ qa][scm-repository-selection-qa] | [qa.md](qa.md) | `custom_properties` |

`scope`, `scm-search` и extension-модификаторы **не** имеют файлов здесь (действуют на
API через mapping).

## Порядок применения

1. После base list и mapping-шагов (dozagрузка) каждая карточка полная для заданных
   критериев.
2. Фильтры `qp`, `qn`, `qt`, `qa` применяются **локально**; несколько условий одного
   флага и разных флагов — **AND**, как в hub.

## Не описывать здесь

- REST-запросы — [mapping/](../repository-selection-mapping/index.md).

<!-- LINKS -->
[scm-repository-selection]: ../../../platform/scm-repository-selection.md
[scm-repository-selection-qp-qn]: ../../../platform/scm-repository-selection.md#qp-и-qn
[scm-repository-selection-qt]: ../../../platform/scm-repository-selection.md#qt-теги
[scm-repository-selection-qa]: ../../../platform/scm-repository-selection.md#qa-атрибуты
[adr-local-eval]: ../../../../../ADR/ADR-26102001-scm-repository-selection-evaluation.md
