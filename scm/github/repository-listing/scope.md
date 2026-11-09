# Листинг `scope` (GitHub)

Базовый list — [base-enumeration](../base-enumeration.md). Продуктовый контракт `scope` —
[hub § scope][scm-repository-selection-scope].

`GET /search/repositories` **не** поддерживает продуктовый контур `affiliated` / `all` как
отдельные query-параметры. Hub-`scope` **не** транслируется в этот list и **не** меняет
сборку `q` ([listing qp](qp.md), [listing qn](qn.md)).

| После резолва | GET-параметр / эффект на `q` |
| ------------- | ---------------------------- |
| `affiliated`  | — (не добавляется)           |
| `all`         | — (не добавляется)           |

Guard affiliation-scope и предупреждения на `github.com` — [ADR-26102002][adr-saas-guard].
Чтобы сузить list по владельцу, задавайте `--qp` с точным login user/org
([base-enumeration § ограничения API][base-enumeration-limits]).

<!-- LINKS -->
[scm-repository-selection-scope]: ../../../platform/scm-repository-selection.md#scope
[adr-saas-guard]: ../../../../../ADR/ADR-26102002-public-saas-scm-affiliation-guard.md
[base-enumeration-limits]: ../base-enumeration.md#ограничения-api
