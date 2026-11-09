# Обогащение `properties` (GitHub)

Дозагрузка поля `properties` после листинга. Смысл `--qa` — [hub § qa][scm-repository-selection-qa];
фрагменты `props.*` в `q` — [listing/qa.md](../repository-listing/qa.md).

## Когда включать

После резолва был **любой** `--qa` (даже когда все условия только с `=`).
Иначе джоб **не** ставим в очередь.

## HTTP {#дозагрузка-properties}

Для **каждой** карточки репозитория в текущем наборе:

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

## Ошибки `GET …/properties/values`

Повторы [`--retries-number`][cli-retries], затем [`--on-error`][cli-on-error]:

| `--on-error` | Действие |
| ------------ | -------- |
| `abort` | Стоп команды |
| `continue` | У репозитория `properties` не меняем (`{}`); дальше |

403 и 404 после повторов — по таблице `--on-error` выше.

## Не в этом файле

- Разбор argv и AND — hub.
- Проверка условий на карточке — [enumeration/qa.md](../repository-enumeration/qa.md).
- Ошибки list Search — [listing/qa.md](../repository-listing/qa.md).
- Порядок фаз и прогресс очереди — [scm/index.md](../../index.md).

<!-- LINKS -->
[scm-repository-selection-qa]: ../../../platform/scm-repository-selection.md#qa-атрибуты
[base-enumeration-uid]: ../base-enumeration.md#uid
[adr-timestamp]: ../../../../../ADR/ADR-26101704-timestamp-formats.md
[cli-retries]: ../../../cli/index.md#повторы-и---on-error
[cli-on-error]: ../../../cli/index.md#--on-error
[github-custom-properties-values]: https://docs.github.com/en/rest/repos/custom-properties?apiVersion=2026-03-10#get-all-custom-property-values-for-a-repository
