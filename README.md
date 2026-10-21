# Спецификации Git Herder

Монорепозиторий прикладных спецификаций **Git Herder** (`git-herder`): контракты
CLI, сквозные механизмы и иные постановки прикладного уровня.

Источник правды для прикладного контракта — здесь, а не в `application`. Порядок
слоёв — [change-flow](../../playbook/workflow/change-flow.md); принципы
[spec-first](../../playbook/principles/spec-first.md) и
[contract-first](../../playbook/principles/contract-first.md).

## Связь с другими репозиториями

Репозиторий стоит **ниже** [замысла продукта][product-intent-readme],
`architecture` и `ADR`, **выше** `application`.

- Каждый документ **ссылается вверх**: [Capability][CAPABILITY-TERM], NFR из
  product-intent, при необходимости — ADR. Правило —
  [ссылки только вверх][upward-references].
- User stories и acceptance criteria — в таск-трекере; здесь — результат
  системного анализа.
- Термины — только в [глоссарии][glossary-index]; в спеках — ссылка.

## Что сюда не кладём

- [Замысел продукта][PRODUCT-INTENT-TERM], ArchiMate, ADR как замена постановке.
- Реализацию и тесты — `application`.

## Структура

```text
specifications/
├── README.md
├── cli/
│   ├── index.md        # глобальные флаги, exit codes, semver CLI
│   └── <command>/      # index.md, flow.md при необходимости
└── platform/           # сквозные NFR (logging, config, …)
```

Шаблон спеки команды:
[cli-command-spec.md](../../playbook/templates/cli-command-spec.md).

## SSOT map

| Тема | SSOT | Статус |
| ---- | ---- | ------ |
| Глобальные флаги CLI, приоритет конфигурации, общие коды завершения | [cli/index.md](cli/index.md) | актуально |
| Контракт команды `<name>` | `cli/<name>/index.md` | по мере появления команд |
| Журнал выполнения, JSON, sinks, redaction | [platform/logging.md](platform/logging.md) | актуально |
| Профили, резолв конфигурации (CLI, YAML, builtin) | [platform/profiles.md](platform/profiles.md) | актуально |
| Критерии SCM-отбора (`--qp`, `--qn`, `--qt`, `--qa`, `--scm-search`, `--limited-scope`) | [platform/scm-repository-selection.md](platform/scm-repository-selection.md) | актуально |
| Контракт команды `profile` | [cli/profile/index.md](cli/profile/index.md) | актуально |

Не дублируйте таблицы глобальных флагов и инварианты платформы вне SSOT —
ссылайтесь на строку таблицы выше.

<!-- LINKS -->
[PRODUCT-INTENT-TERM]: ../../glossary/terms/product-intent.md
[CAPABILITY-TERM]: ../../glossary/terms/capability.md
[upward-references]: ../../playbook/principles/upward-references.md
[glossary-index]: ../../glossary/index.md
[product-intent-readme]: ../product-intent/README.md
