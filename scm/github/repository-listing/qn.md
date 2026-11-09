# Листинг `qn` (GitHub)

Смысл флага — [hub § qp/qn][scm-repository-selection-qp-qn]. Общий `q` —
[index § сборка q](index.md#сборка-q).

На GitHub `--qn` **только** в Search. Повторной фильтрации по `name` нет — файла в
[repository-enumeration/](../repository-enumeration/index.md) для `qn` нет.

## Когда есть смысл

Хотя бы один `--qn` после резолва. Иначе фрагментов отсюда нет.

## Что делаем

1. Каждое значение `--qn` проверяем так же, как `--qp`
   ([qp.md § что делаем][qp-algorithm], [base-enumeration § ограничения API][base-enumeration-limits]).
2. Каждое прошедшее проверку значение — **отдельный** кусок `q` **без** префикса
   (`in:name` не используем).
3. Несколько `--qn`: куски склеиваем через `+` между собой и с [qp](qp.md) /
   [qa](qa.md) ([index](index.md#сборка-q)).

### Пример

```text
--qn code2 --qn archi
```

→ в `q`: `code2+archi`

→ `GET /search/repositories?q=code2+archi` (в URL кодируем).

## Не в этом файле

- Локальная проверка `name` — не делается.

<!-- LINKS -->
[scm-repository-selection-qp-qn]: ../../../platform/scm-repository-selection.md#qp-и-qn
[base-enumeration-limits]: ../base-enumeration.md#ограничения-api
[qp-algorithm]: qp.md#что-делаем
