# Обогащение `properties` (GitLab)

Дозагрузка поля `properties` после листинга. Смысл `--qa` — [hub § qa][scm-repository-selection-qa];
server-side `custom_attributes` на list — [listing/qa.md](../repository-listing/qa.md).

## Когда включать

После резолва был хотя бы один `--qa`. Иначе этот джоб **не** ставим в очередь.

## HTTP

Для **каждой** карточки проекта в текущем наборе выполнить шаги 1–5 ниже.

1. Отправляет запрос `GET /api/v4/projects/:id/custom_attributes`, подставив в
   `:id` числовой `uid` из карточки этого проекта.
2. Получает JSON-массив записей; у каждой записи в API есть строковые поля
   `key` (имя атрибута) и `value` (значение как текст).
3. Разбирает массив с первого элемента к последнему и собирает объект
   «имя → значение» для карточки. Если одно и то же имя встретилось несколько
   раз, оставляет значение из последней записи с этим именем.
4. Для каждой пары «имя → текст из API» определяет тип в карточке в таком
   порядке (как только правило сработало — дальше не смотрит):
   - если текст без учёта регистра равен `true` или `false` — записывает
     boolean;
   - иначе, если текст читается как число JSON (без экспоненты; целые без
     `.0`) — записывает number;
   - иначе, если текст приведёт к профилю data-timestamp
     ([ADR-26101704][adr-timestamp]) — записывает date (строка в JSON
     карточки);
   - иначе — записывает string (UTF-8 как в ответе API).
5. Заменяет поле `properties` в карточке этим объектом целиком (значение `{}`
   из ответа list **не** сохраняет).

Если API вернул пустой массив, кладёт в карточку `properties` = `{}`.
Как потом сравнивать значения при `--qa` — только
[карточка § qa][scm-repository-card-qa].

## Ошибки `GET …/custom_attributes`

Повторы — [`--retries-number`][cli-retries], затем [`--on-error`][cli-on-error]:

| `--on-error` | Действие |
| ------------ | -------- |
| `abort` | Стоп команды |
| `continue` | У этого проекта `properties` не трогаем (остаётся `{}`); идём дальше |

## Не здесь

- Разбор argv, операторы, AND — hub.
- Проверка условий на карточке — [enumeration/qa.md](../repository-enumeration/qa.md).
- Порядок фаз и прогресс очереди — [scm/index.md](../../index.md).

<!-- LINKS -->
[scm-repository-selection-qa]: ../../../platform/scm-repository-selection.md#qa-атрибуты
[scm-repository-card-qa]: ../../../platform/scm-repository-card.md#qa-и-properties
[adr-timestamp]: ../../../../../ADR/ADR-26101704-timestamp-formats.md
[cli-retries]: ../../../cli/index.md#повторы-и---on-error
[cli-on-error]: ../../../cli/index.md#--on-error
