# Маппинг `qt` (GitHub)

## Контракт

[hub § qt][scm-repository-selection-qt] — какие условия `--qt` заданы после резолва.

## Запросы

Дополнительные HTTP-вызовы **per repository** (или batch, если API позволяет), чтобы
наполнить в карточке поля для topics/labels ([base-enumeration.md](../base-enumeration.md)).

Пример: для `--qt team=Alpha` адаптер должен получить данные, достаточные для проверки
`team=Alpha` на клиенте ([enumeration/qt.md](../repository-enumeration/qt.md)).

## Параметры и заголовки

_(Эндпоинты topics, labels и т.д. — по мере фиксации.)_

## Комбинация

Запросы enrich выполняются **после** base list с учётом [scope](scope.md) на этапах,
где scope применяется к тем же сессиям API.

## Не описывать здесь

- Сопоставление `key=value` на карточке — [enumeration/qt.md](../repository-enumeration/qt.md).

<!-- LINKS -->
[scm-repository-selection-qt]: ../../../platform/scm-repository-selection.md#qt-теги-топики-лейблы
