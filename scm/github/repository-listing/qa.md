# Листинг `qa` (GitHub)

Server-side list Search. Смысл флага и разбор argv —
[hub § qa][scm-repository-selection-qa]. Условия с операторами кроме `=` —
[enumeration/qa.md](../repository-enumeration/qa.md). Дозагрузка `properties` —
[enrichment/qa.md](../repository-enrichment/qa.md).

## Когда есть смысл

Хотя бы один `--qa` после резолва. Иначе шаги ниже не делаем; `properties` в карточке
остаётся `{}` ([base-enumeration § карточка][base-enumeration-card]).

## Равенства в `q` (оператор `=`)

Только `--qa` с `=` (как в hub) дают куски для общей `q`
([index § сборка q](index.md#сборка-q)):

1. Взять key и value из каждого такого флага.
2. Кусок: `props.<key>:<value>`. В key/value регэкп не нужен; символы шире, чем у
   `--qp`/`--qn`, **можно**.
3. Несколько равенств: все куски через `+` в одной `q` вместе с [qp](qp.md) и
   [qn](qn.md).

Пример: `--qa foo=bar --qa tier=gold` → `props.foo:bar+props.tier:gold` (порядок
не важен).

`!=`, `<`, `>`, `<=`, `>=` в `q` **не** пишем — проверка на клиенте —
[enumeration/qa.md](../repository-enumeration/qa.md).

## Ошибки list (в т.ч. `q` с `props.…`)

Сначала повторы [`--retries-number`][cli-retries], потом:

| Ситуация | Действие |
| -------- | -------- |
| **403** после повторов | Стоп; в тексте — про права Search / custom properties |
| Другая ошибка list | `abort` — стоп; `continue` — **пустой** набор карточек |

## Не в этом файле

- Разбор argv и AND — hub.
- Проверка условий на карточке — [enumeration/qa.md](../repository-enumeration/qa.md).
- Дозагрузка `properties` — [enrichment/qa.md](../repository-enrichment/qa.md).

<!-- LINKS -->
[scm-repository-selection-qa]: ../../../platform/scm-repository-selection.md#qa-атрибуты
[base-enumeration-card]: ../base-enumeration.md#карточка-репозитория
[cli-retries]: ../../../cli/index.md#повторы-и---on-error
