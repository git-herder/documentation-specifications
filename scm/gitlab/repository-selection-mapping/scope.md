# Маппинг `scope` (GitLab)

## Контракт

[hub § scope][scm-repository-selection-scope].

## Запросы

[base-enumeration.md](../base-enumeration.md); режим `--q*`, не `--scm-search`.

## Параметры

| `scope` | На API |
| ------- | ------ |
| `affiliated` | **membership** — [base-enumeration § membership](../base-enumeration.md#query-membership-scope) |
| `all` | Без membership |

## Комбинация

На list с [include-inactive](include-inactive.md): оба query применяются к одному
запросу; детали `active` — в include-inactive, membership — здесь.

## Условия хоста

[ADR-26102002][adr-saas-guard], [SaaS scope][adr-saas-guard-scope].

<!-- LINKS -->
[scm-repository-selection-scope]: ../../../platform/scm-repository-selection.md#scope
[adr-saas-guard]: ../../../../../ADR/ADR-26102002-public-saas-scm-affiliation-guard.md
[adr-saas-guard-scope]: ../../../../../ADR/ADR-26102002-public-saas-scm-affiliation-guard.md#scope-public-saas
