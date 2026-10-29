# Реестр маппинга SCM-отбора (GitHub)

SSOT: как параметры отбора (hub и extension) → **HTTP** (эндпоинты, query, body,
заголовки) и как опции **комбинируются** на стороне API.

Контракт CLI — [hub][scm-repository-selection]; extension —
[options](../repository-selection-options.md). Базовый list и карточка —
[base-enumeration.md](../base-enumeration.md). Клиентская фильтрация —
[repository-enumeration/](../repository-enumeration/index.md).

## Реестр параметров (REST)

| Параметр | Контракт (SSOT) | Mapping | Примечание |
| -------- | --------------- | ------- | ---------- |
| `scope` | [hub § scope][scm-repository-selection-scope] | [scope.md](scope.md) | Query на list/enrich |
| `scm-search` | [hub § scm-search][scm-repository-selection-scm-search] | [scm-search.md](scm-search.md) | Отдельная ветка API |
| `qt` | [hub § qt][scm-repository-selection-qt] | [qt.md](qt.md) | Только дозагрузка в карточку |
| `qa` | [hub § qa][scm-repository-selection-qa] | [qa.md](qa.md) | Только дозагрузка |

`qp` и `qn` **не** имеют mapping (нет REST). Фильтрация — [enumeration/qp.md](../repository-enumeration/qp.md),
[enumeration/qn.md](../repository-enumeration/qn.md).

## Комбинации на одном запросе

Если несколько модификаторов задают query на **одном** базовом list-запросе, SSOT
комбинации — в [scope.md](scope.md) (ссылки на другие mapping-файлы, без дублирования
таблиц).

<!-- LINKS -->
[scm-repository-selection]: ../../../platform/scm-repository-selection.md
[scm-repository-selection-scope]: ../../../platform/scm-repository-selection.md#scope
[scm-repository-selection-scm-search]: ../../../platform/scm-repository-selection.md#scm-search
[scm-repository-selection-qt]: ../../../platform/scm-repository-selection.md#qt-теги-топики-лейблы
[scm-repository-selection-qa]: ../../../platform/scm-repository-selection.md#qa-атрибуты
