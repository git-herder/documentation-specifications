# Реестр листинга SCM-отбора (GitHub)

CLI → HTTP (server-side list). [base-enumeration.md](../base-enumeration.md),
[repository-enrichment/](../repository-enrichment/index.md),
[repository-enumeration/](../repository-enumeration/index.md),
[options](../repository-selection-options.md), [hub][scm-repository-selection].

## Реестр (REST list)

| Параметр | Контракт | Листинг |
| -------- | -------- | ------- |
| `scope` | [hub § scope][scm-repository-selection-scope] | [scope.md](scope.md) |
| `qp` | [hub § qp/qn][scm-repository-selection-qp-qn] | [qp.md](qp.md) |
| `qn` | [hub § qp/qn][scm-repository-selection-qp-qn] | [qn.md](qn.md) |
| `qt` | [hub § qt][scm-repository-selection-qt] | [qt.md](qt.md) |
| `qa` | [hub § qa][scm-repository-selection-qa] | [qa.md](qa.md) |

`scm-search` — отдельная ветка hub; в режиме `--q*` не комбинируется с переносимыми
критериями.

Обогащение карточек после листинга — [repository-enrichment/index.md](../repository-enrichment/index.md).

## Сборка query `q` {#сборка-q}

SSOT строки `q` для базового list `GET /search/repositories`
([base-enumeration](../base-enumeration.md)). Участвуют только server-side фрагменты:

| Источник | Когда даёт фрагменты |
| -------- | -------------------- |
| [qp.md](qp.md) | После резолва есть хотя бы один `--qp` |
| [qn.md](qn.md) | После резолва есть хотя бы один `--qn` |
| [qa.md](qa.md) | После резолва есть `--qa` с оператором `=` (разбор — hub) |
| [qt.md](qt.md) | **Не** добавляет фрагментов в `q` |

### Алгоритм

1. Для каждого `--qp` и `--qn` — валидация и фрагменты по [qp.md](qp.md) /
   [qn.md](qn.md); несоответствие алфавиту — exit `1` до HTTP
   ([base-enumeration § ограничения API][base-enumeration-limits]).
2. Для каждого `--qa` с `=` — фрагменты `props.<key>:<value>` по [qa.md](qa.md).
3. Собрать **одну** строку: все фрагменты из шагов 1–2, конкатенация через символ
   `+` (AND в синтаксисе GitHub Search). Порядок фрагментов и видов `--q*` **не**
   нормативен.
4. Если после шагов 1–2 **нет ни одного** фрагмента (например, только `--qt`, или
   только `--qa` без `=`) — **ошибка до HTTP**, exit `1`
   ([hub § ошибки][scm-repository-selection-errors]).
5. Значение `q` в query string кодирует по правилам HTTP (percent-encoding).

Пример: `--qn code2 --qn archi --qp jopos --qa foo=bar` → фрагменты из qn, qp, qa;
итоговая строка эквивалентна `code2+archi+org:jopos+user:jopos+props.foo:bar` (порядок
слагаемых может отличаться).

Лимит выдачи Search API (~1000 результатов) — поведение GitHub;
[документация search/repositories][github-search-repositories].

`scope` на `q` и базовый list **не** влияет — [scope.md](scope.md).

## Не здесь

- Семантика argv, AND между `--q*`, разбор `--qt` / `--qa` — hub.
- Поля карточки — [scm-repository-card][scm-repository-card], наполнение из list —
  [base-enumeration § карточка][base-enumeration-card].
- Порядок фаз listing → enrichment — [scm/index.md](../../index.md).

<!-- LINKS -->
[scm-repository-selection]: ../../../platform/scm-repository-selection.md
[scm-repository-selection-scope]: ../../../platform/scm-repository-selection.md#scope
[scm-repository-selection-qp-qn]: ../../../platform/scm-repository-selection.md#qp-и-qn
[scm-repository-selection-qt]: ../../../platform/scm-repository-selection.md#qt-теги
[scm-repository-selection-qa]: ../../../platform/scm-repository-selection.md#qa-атрибуты
[scm-repository-selection-errors]: ../../../platform/scm-repository-selection.md#ошибки-и-коды-выхода
[scm-repository-card]: ../../../platform/scm-repository-card.md
[base-enumeration-limits]: ../base-enumeration.md#ограничения-api
[base-enumeration-card]: ../base-enumeration.md#карточка-репозитория
[github-search-repositories]: https://docs.github.com/en/rest/search/search?apiVersion=2026-03-10#search-repositories
