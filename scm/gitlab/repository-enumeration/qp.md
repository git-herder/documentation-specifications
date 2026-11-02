# Клиентская фильтрация `qp` (GitLab)

## Контракт

[hub § qp и qn][scm-repository-selection-qp-qn]. Проверяем `path` карточки
([scm-repository-card][scm-repository-card]; наполнение path для GitLab —
[base-enumeration](../base-enumeration.md#карточка-репозитория)).

Запросов к API нет — только фильтр по уже собранным карточкам.

## Данные на карточке

Поле **`path`**: [base-enumeration § карточка](../base-enumeration.md#карточка-репозитория).

При резолве **`qt`**: **`labels`** — [mapping/qt.md](../repository-selection-mapping/qt.md);
иначе из `topics` в base-enumeration § карточка.

При резолве **`qa`**: **`properties`** — [mapping/qa.md](../repository-selection-mapping/qa.md);
иначе `{}` как в [base-enumeration § карточка](../base-enumeration.md#карточка-репозитория).

## Алгоритм

Для каждого `--qp` после резолва:

1. Собрать регэкп (re2) — правила в hub.
2. Регэкп **собрался** — оставить репозиторий, если выражение **находит совпадение**
   в строке `path` (как есть, без доп. нормализации).
3. **Не собрался** — hub: считать условие проверкой **вхождения подстроки** в `path`.

Несколько `--qp` — **AND** друг с другом и с `--qn`, `--qt`, `--qa`.

Подробности регэкп / подстроки — только hub, здесь не повторяем.

<!-- LINKS -->
[scm-repository-selection-qp-qn]: ../../../platform/scm-repository-selection.md#qp-и-qn
[scm-repository-card]: ../../../platform/scm-repository-card.md
