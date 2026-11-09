# Листинг `qa` (GitLab)

CLI → HTTP (server-side list). Смысл `--qa` и разбор argv —
[hub § qa][scm-repository-selection-qa]. Отбор на клиенте (все операторы, кроме
`=`) — [enumeration/qa.md](../repository-enumeration/qa.md). Дозагрузка
`properties` — [enrichment/qa.md](../repository-enrichment/qa.md).

## Когда включать

После резолва argv и профиля задан хотя бы один `--qa`. Иначе шаги ниже не
выполняем; в карточке `properties` = `{}`, как из ответа list в
[base-enumeration § карточка](../base-enumeration.md#карточка-репозитория).

## List проектов

Базовый запрос — `GET /api/v4/projects` ([base-enumeration](../base-enumeration.md)).
На каждую страницу добавляем [scope](scope.md) и [include-inactive](include-inactive.md).

| Условия `--qa` после резолва | Query |
| -------------------------- | ----- |
| Нет ни одного с оператором `=` (разбор — hub) | Параметры `custom_attributes[…]` не добавляем |
| Есть `=` (одно или несколько) | На каждое равенство: `custom_attributes[<key>]=<value>` |

Если в отборе есть условия «атрибут **равен** значению» (разбор флагов — hub):

1. Берёт из каждого такого `--qa` имя атрибута и значение.
2. На **каждый** запрос очередной страницы списка проектов добавляет к уже
   заданным параметрам (`scope`, `include-inactive` и т.д.) пары «имя атрибута →
   значение» в виде `custom_attributes[имя]=значение`. Спецсимволы в имени и
   значении кодирует так, как требует HTTP для строки запроса.
3. Несколько равенств задаёт **в одном** запросе страницы: все пары через `&`.
   Пример двух условий: `--qa foo=bar --qa bar=buzz` → в URL одновременно
   `custom_attributes[foo]=bar` и `custom_attributes[bar]=buzz`. GitLab должен
   вернуть только проекты, у которых **одновременно** выполняются все эти
   равенства (логика «и» на стороне сервера).

Условия с `!=`, `<`, `>`, `<=`, `>=` на этом шаге **не** использует: список
проектов запрашивает без них; проверка на клиенте —
[enumeration/qa.md](../repository-enumeration/qa.md).

По документации GitLab такой отбор в list часто доступен только **администратору**
инстанса. Git Herder роль токена заранее не проверяет: если пользователь задал
равенства в `--qa`, параметры в запрос list **всё равно** добавляет; при отказе
(например 403) — см. раздел «Ошибки list» ниже.

## Ошибки list (в том числе query `custom_attributes`)

Повторы — [`--retries-number`][cli-retries], дальше:

| Ситуация | Действие |
| -------- | -------- |
| **403** после всех повторов | Стоп команды; в сообщении — про custom attributes и права на list |
| Любая другая ошибка list после повторов | `abort` ([`--on-error`][cli-on-error]) — стоп; `continue` — **пустой** набор карточек |

Повторы — [cli/index.md][cli-retries].

## Не здесь

- Разбор argv, операторы, AND — hub.
- Проверка условий на карточке — [enumeration/qa.md](../repository-enumeration/qa.md).
- Дозагрузка `properties` — [enrichment/qa.md](../repository-enrichment/qa.md).

<!-- LINKS -->
[scm-repository-selection-qa]: ../../../platform/scm-repository-selection.md#qa-атрибуты
[cli-retries]: ../../../cli/index.md#повторы-и---on-error
[cli-on-error]: ../../../cli/index.md#--on-error
