# Базовое перечисление (GitLab)

SSOT: базовый list, пагинация, **карточка** GitLab (без cross-SCM обобщения).

Mapping — [repository-selection-mapping/](repository-selection-mapping/index.md).
Клиентская фильтрация — [repository-enumeration/](repository-enumeration/index.md).

## Базовый list

_(Эндпоинт(ы) — по мере фиксации. Query от `scope`, `include-inactive` — mapping.)_

## Пагинация

_(По мере фиксации.)_

## Карточка репозитория

| Поле | Назначение |
| ---- | ---------- |
| `path` | Путь (namespace/project) — [enumeration/qp.md](repository-enumeration/qp.md) |
| `name` | Имя — [enumeration/qn.md](repository-enumeration/qn.md) |
| `tags`, `labels` | Для [enumeration/qt.md](repository-enumeration/qt.md) |
| `custom_properties` | Для [enumeration/qa.md](repository-enumeration/qa.md) |

## Query membership (scope) {#query-membership-scope}

REST для **membership** при `scope=affiliated` — SSOT здесь;
[mapping/scope.md](repository-selection-mapping/scope.md).

## Query active (include-inactive) {#query-active-filter}

REST для `active=true` при default `include-inactive` — SSOT здесь;
[mapping/include-inactive.md](repository-selection-mapping/include-inactive.md).

<!-- LINKS -->
[scm-repository-selection]: ../../platform/scm-repository-selection.md
