# Клиентская фильтрация `qt` (GitLab)

## Контракт

[hub § qt][scm-repository-selection-qt]. Смотрим поле `labels` карточки
[scm-repository-card][scm-repository-card].

**Наличие** (`--qt имя` без `=null`) уже отфильтровано на list — параметр `topic` в
[mapping/qt.md](../repository-selection-mapping/qt.md). Здесь только **отсутствие**
(`--qt имя=null`).

## Данные на карточке

Поле **`labels`**: при резолве `qt` — [mapping/qt.md](../repository-selection-mapping/qt.md);
иначе из `topics` в [base-enumeration § карточка](../base-enumeration.md#карточка-репозитория).

## Алгоритм

Для каждого `--qt` после резолва (как разбирать значение — hub § qt):

| Вид условия | На клиенте |
| ----------- | ---------- |
| Наличие | Ничего не проверять |
| Отсутствие (`=null`) | Пропустить репозиторий, если *имя* **есть** в `labels`; иначе оставить |

Сравнение *имени* с элементами `labels` — **точное**, с учётом регистра.

Несколько `--qt` между собой — **AND** (hub). То же AND с `--qp`, `--qn`, `--qa`.

### Пример

```text
--qt foo --qt bar=null
```

List с `topic`, где есть urlencoded `foo`, и в `labels` нет строки `bar`.

<!-- LINKS -->
[scm-repository-selection-qt]: ../../../platform/scm-repository-selection.md#qt-теги
[scm-repository-card]: ../../../platform/scm-repository-card.md
