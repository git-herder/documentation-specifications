# Клиентская фильтрация `qa` (GitHub)

## Контракт

Операторы и лексикографическое сравнение — [hub § qa][scm-repository-selection-qa].

## Вход

`custom_properties` ([base-enumeration.md](../base-enumeration.md)).

## Алгоритм

Для каждого `--qa`: проверка по hub (ключ обязан существовать для `!=`; сравнение
строк). Несколько `--qa` — AND.

Дозагрузка — [mapping/qa.md](../repository-selection-mapping/qa.md).

<!-- LINKS -->
[scm-repository-selection-qa]: ../../../platform/scm-repository-selection.md#qa-атрибуты
