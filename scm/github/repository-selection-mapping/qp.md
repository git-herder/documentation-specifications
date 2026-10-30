# Маппинг `qp` (GitHub)

Смысл флага — [hub § qp/qn][scm-repository-selection-qp-qn]. Как собирается общий
`q` — [index § сборка q](index.md#сборка-q).

На GitHub `--qp` попадает **только** в Search (`q`). Повторной фильтрации по `path`
нет — файла в [repository-enumeration/](../repository-enumeration/index.md) для `qp`
нет.

## Когда есть смысл

Хотя бы один `--qp` после резолва argv и профиля. Иначе фрагментов для `q` отсюда нет.

## Что делаем

Для **каждого** `--qp`:

1. Проверить строку: только `A–Z`, `a–z`, `0–9`, `_`, `-`. Регэкп нельзя. Ошибка —
   exit `1` **до** HTTP ([base-enumeration § ограничения API][base-enumeration-limits]).
2. Из значения сделать фрагмент: `org:<значение>+user:<значение>`.
3. Несколько `--qp`: все такие фрагменты идут в общую `q` через `+` вместе с
   [qn](qn.md) и [qa](qa.md) ([index](index.md#сборка-q)).

### Пример

```text
--qp jopos --qp cragent
```

→ в `q`: `org:jopos+user:jopos+org:cragent+user:cragent`

→ `GET /search/repositories?q=…` (значение в URL кодируем).

## Не в этом файле

- AND между разными `--q*` — hub.
- Проверка `path` на карточке — не делается.

<!-- LINKS -->
[scm-repository-selection-qp-qn]: ../../../platform/scm-repository-selection.md#qp-и-qn
[base-enumeration-limits]: ../base-enumeration.md#ограничения-api
