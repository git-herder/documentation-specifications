# Клиентская фильтрация SCM-отбора (GitHub)

[base-enumeration.md](../base-enumeration.md), [hub][scm-repository-selection],
[mapping/](../repository-selection-mapping/index.md), [ADR-26102001][adr-local-eval].

## Назначение каталога

Здесь лежат **только** те опции `--q*`, для которых после list нужна **локальная**
фильтрация карточек. Один параметр — один файл.

Если опции **нет** в этом каталоге (и в реестре ниже), для GitHub она отрабатывает
**только на стороне Search** — см. [repository-selection-mapping/](../repository-selection-mapping/index.md)
(`qp`, `qn`, равенства `qa` в `q` и т.д.).

## Реестр

| Параметр | Hub | Файл | Поле карточки |
| -------- | --- | ---- | ------------- |
| `qt` | [§ qt][scm-repository-selection-qt] | [qt.md](qt.md) | `labels` |
| `qa` | [§ qa][scm-repository-selection-qa] | [qa.md](qa.md) | `properties` (операторы кроме `=`) |

`qp`, `qn` — только [mapping](../repository-selection-mapping/index.md#сборка-q), файлов
enumeration **нет**.

`scope`, `on-limit-exceeded`, `scm-search` — mapping / options, не клиентская фильтрация.

## Порядок

После полного list, постраничной догрузки `properties` для `--qa` ([mapping/qa.md](../repository-selection-mapping/qa.md))
— фильтры из файлов выше; между условиями **AND** (hub).

<!-- LINKS -->
[scm-repository-selection]: ../../../platform/scm-repository-selection.md
[scm-repository-selection-qt]: ../../../platform/scm-repository-selection.md#qt-теги
[scm-repository-selection-qa]: ../../../platform/scm-repository-selection.md#qa-атрибуты
[adr-local-eval]: ../../../../../ADR/ADR-26102001-scm-repository-selection-evaluation.md
