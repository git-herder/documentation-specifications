# Маппинг `qa` (GitHub)

Смысл флага и разбор argv — [hub § qa][scm-repository-selection-qa]. Условия с
операторами кроме `=` — [enumeration/qa.md](../repository-enumeration/qa.md).

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

`!=`, `<`, `>`, `<=`, `>=` в `q` **не** пишем — проверяем после догрузки
properties.

## Догрузка `properties` {#дозагрузка-properties}

Если после резолва был **любой** `--qa` (даже когда все условия только с `=`).

На **каждой** странице list, **до** запроса следующей страницы, для **каждого**
репозитория из `items` этой страницы:

1. Из `uid` карточки (`owner/repo`, [base-enumeration § uid][base-enumeration-uid])
   выделить `owner` и `repo` (первый `/`). Битый `uid` → поведение
   [`--on-error`][cli-on-error] = `abort`.
2. `GET /repos/{owner}/{repo}/properties/values`
   ([REST custom properties][github-custom-properties-values]).
3. Ответ — массив JSON; у элемента есть `property_name` и `value`.
4. Собрать объект для поля `properties`:
   - идём по массиву с начала; при повторе имени берём **последнее** значение;
   - тип value в карточке (первое подошедшее правило):
     - `null` — ключ не добавляем;
     - boolean → boolean;
     - number (без экспоненты; целые без `.0`) → number;
     - string, подходящая под data-timestamp ([ADR-26101704][adr-timestamp]) → date;
     - массив → string (компактный JSON без пробелов после `,` и `:`);
     - остальное → string (`JSON.stringify` без пробелов).
5. Поле `properties` карточки **полностью** заменить этим объектом.

Пустой массив → `properties` = `{}`.

## Ошибки list (в т.ч. `q` с `props.…`)

Сначала повторы [`--retries-number`][cli-retries], потом:

| Ситуация | Действие |
| -------- | -------- |
| **403** после повторов | Стоп; в тексте — про права Search / custom properties |
| Другая ошибка list | `abort` — стоп; `continue` — пустой набор, догрузку не делаем |

## Ошибки `GET …/properties/values`

Повторы [`--retries-number`][cli-retries], затем [`--on-error`][cli-on-error]:

| `--on-error` | Действие |
| ------------ | -------- |
| `abort` | Стоп команды |
| `continue` | У репозитория `properties` не меняем (`{}`); дальше |

403 и 404 после повторов — по таблице `--on-error` выше.

## Порядок

1. List с готовой `q` ([index](index.md#сборка-q)).
2. На каждой странице — догрузка properties (этот раздел), если был `qa`.
3. Локальные фильтры — [enumeration/](../repository-enumeration/index.md).

## Не в этом файле

- Разбор argv и AND — hub.
- Проверка условий на карточке — [enumeration/qa.md](../repository-enumeration/qa.md).

<!-- LINKS -->
[scm-repository-selection-qa]: ../../../platform/scm-repository-selection.md#qa-атрибуты
[base-enumeration-card]: ../base-enumeration.md#карточка-репозитория
[base-enumeration-uid]: ../base-enumeration.md#uid
[adr-timestamp]: ../../../../../ADR/ADR-26101704-timestamp-formats.md
[cli-retries]: ../../../cli/index.md#повторы-и---on-error
[cli-on-error]: ../../../cli/index.md#--on-error
[github-custom-properties-values]: https://docs.github.com/en/rest/repos/custom-properties?apiVersion=2026-03-10#get-all-custom-property-values-for-a-repository
