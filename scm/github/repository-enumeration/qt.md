# Клиентская фильтрация `qt` (GitHub)

## Контракт

Операторы `=` / `!=`, OR внутри одного условия между реализациями класса — [hub § qt][scm-repository-selection-qt].

## Вход

Поля карточки для тегов/топиков/лейблов — [base-enumeration.md](../base-enumeration.md)
(`topics`, `labels`, …).

## Алгоритм

Для одного `--qt key=value` (или `!=`): условие выполнено, если **любая** нативная
реализация на GitHub (topic, label, …) даёт совпадение по правилам hub. Несколько
`--qt` — AND между условиями.

Дозагрузка данных в карточку — [mapping/qt.md](../repository-selection-mapping/qt.md).

## Не описывать здесь

- Какие HTTP-запросы наполняют topics/labels — mapping.

<!-- LINKS -->
[scm-repository-selection-qt]: ../../../platform/scm-repository-selection.md#qt-теги-топики-лейблы
