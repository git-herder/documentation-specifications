# Маппинг `include-inactive` (GitLab)

## Контракт

[options § include-inactive][options-include-inactive].

## Запросы

Все HTTP list/enrich в режиме `--q*` ([base-enumeration.md](../base-enumeration.md)).
Не при `--scm-search`.

## Параметры

| После резолва | На API |
| ------------- | ------ |
| `false` (default) | `active=true` — [base-enumeration § active](../base-enumeration.md#query-active-filter) |
| `true` | Условие не добавляется |

## Комбинация

С [scope](scope.md) на одном list-запросе.

<!-- LINKS -->
[options-include-inactive]: ../../repository-selection-options.md#include-inactive
