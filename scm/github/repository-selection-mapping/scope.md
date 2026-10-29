# Маппинг `scope` (GitHub)

## Контракт

[hub § scope][scm-repository-selection-scope].

## Запросы

Базовый list и enrich — [base-enumeration.md](../base-enumeration.md). Модификатор
влияет на query/path **всех** HTTP-шагов отбора в режиме `--q*` (не `--scm-search`).

## Параметры и заголовки

| `scope` после резолва | На API |
| --------------------- | ------ |
| `affiliated` | Продуктовая семантика **involves**; имена query — [base-enumeration § involves](../base-enumeration.md#query-involves-scope) |
| `all` | Ограничение **involves** **не** добавляется |

## Комбинация с другими опциями

На одном list-запросе может сочетаться только с параметрами, описанными для того же
upstream в mapping (P0: иных extension-флагов нет).

## Условия хоста

[ADR-26102002][adr-saas-guard], [scope на SaaS][adr-saas-guard-scope] для `github.com`.

## Не описывать здесь

- Клиентскую фильтрацию — [repository-enumeration/](../repository-enumeration/index.md).

<!-- LINKS -->
[scm-repository-selection-scope]: ../../../platform/scm-repository-selection.md#scope
[adr-saas-guard]: ../../../../../ADR/ADR-26102002-public-saas-scm-affiliation-guard.md
[adr-saas-guard-scope]: ../../../../../ADR/ADR-26102002-public-saas-scm-affiliation-guard.md#scope-public-saas
