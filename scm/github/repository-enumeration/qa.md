# Клиентская фильтрация `qa` (GitHub)

## Контракт

Операторы и AND — [hub § qa][scm-repository-selection-qa]. Данные — поле
`properties` ([scm-repository-card][scm-repository-card]); как сравнивать строки —
[карточка § qa][scm-repository-card-qa].

Условия с **`=`** уже отфильтрованы на list через `q`
([mapping/qa.md](../repository-selection-mapping/qa.md)); на клиенте их **не**
проверяем повторно.

## Вход

Карточки после полного list и постраничного enrich по
[mapping/qa.md](../repository-selection-mapping/qa.md), если после резолва был `qa`.

## Алгоритм

Для каждого `--qa` после резолва (key, оператор, value — hub):

| Оператор | Действие |
| -------- | -------- |
| `=` | Пропуск |
| `!=`, `<`, `>`, `<=`, `>=` | Оставить репо, только если условие выполняется (hub + канонические строки карточки) |

Нет ключа в `properties` → условие **не выполняется** (и для `!=` тоже: hub
требует, чтобы ключ был).

Все `--qa` — **AND** между собой и с `--qt` (hub).

<!-- LINKS -->
[scm-repository-selection-qa]: ../../../platform/scm-repository-selection.md#qa-атрибуты
[scm-repository-card]: ../../../platform/scm-repository-card.md
[scm-repository-card-qa]: ../../../platform/scm-repository-card.md#qa-и-properties
