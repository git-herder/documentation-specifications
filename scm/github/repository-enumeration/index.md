# Клиентская фильтрация SCM-отбора (GitHub)

[base-enumeration.md](../base-enumeration.md), [hub][scm-repository-selection],
[listing/](../repository-listing/index.md),
[enrichment/](../repository-enrichment/index.md), [ADR-26102001][adr-local-eval].

## Назначение каталога

Здесь лежат **только** те опции `--q*`, для которых часть критерия проверяется
**локально** на карточке. Один параметр — один файл.

Если опции **нет** в этом каталоге (и в реестре ниже), для GitHub она отрабатывает
**только на стороне Search** — см. [repository-listing/](../repository-listing/index.md)
(`qp`, `qn`, равенства `qa` в `q` и т.д.).

## Реестр

| Параметр | Hub | Файл | Поле карточки |
| -------- | --- | ---- | ------------- |
| `qt` | [§ qt][scm-repository-selection-qt] | [qt.md](qt.md) | `labels` |
| `qa` | [§ qa][scm-repository-selection-qa] | [qa.md](qa.md) | `properties` (операторы кроме `=`) |

`qp`, `qn` — только [listing](../repository-listing/index.md#сборка-q), файлов
enumeration **нет**.

`scope`, `on-limit-exceeded`, `scm-search` — listing / options, не клиентская фильтрация.

Композиция условий между файлами каталога — **AND** ([hub][scm-repository-selection]).

<!-- LINKS -->
[scm-repository-selection]: ../../../platform/scm-repository-selection.md
[scm-repository-selection-qt]: ../../../platform/scm-repository-selection.md#qt-теги
[scm-repository-selection-qa]: ../../../platform/scm-repository-selection.md#qa-атрибуты
[adr-local-eval]: ../../../../../ADR/ADR-26102001-scm-repository-selection-evaluation.md
