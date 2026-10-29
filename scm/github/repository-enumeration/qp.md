# Клиентская фильтрация `qp` (GitHub)

## Контракт

[hub § qp и qn][scm-repository-selection-qp-qn] (регэкп / подстрока, несколько `--qp`
→ AND).

## Вход

Карточка из [base-enumeration.md](../base-enumeration.md); поле **`path`**.

## Алгоритм

Для **каждого** значения `--qp` после резолва:

1. Попытка компиляции как регэкп (re2); при успехе — match по `path`.
2. Иначе — вхождение подстроки в `path`.

Репозиторий проходит, если удовлетворяет **всем** условиям `qp` и остальным
критериям (AND с [qn](qn.md), [qt](qt.md), [qa](qa.md)).

## Не описывать здесь

- REST — mapping не используется для `qp`.

<!-- LINKS -->
[scm-repository-selection-qp-qn]: ../../../platform/scm-repository-selection.md#qp-и-qn
