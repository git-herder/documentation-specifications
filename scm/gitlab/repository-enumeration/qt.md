# Клиентская фильтрация `qt` (GitLab)

## Контракт

[hub § qt][scm-repository-selection-qt] (OR tag/label внутри одного условия — hub).

## Вход

`tags`, `labels` — [base-enumeration.md](../base-enumeration.md).

## Алгоритм

Пример hub: `--qt team=Alpha` ⇔ `(tag team=Alpha) OR (label team=Alpha)` на полях
карточки. Несколько `--qt` — AND.

REST — [mapping/qt.md](../repository-selection-mapping/qt.md).

<!-- LINKS -->
[scm-repository-selection-qt]: ../../../platform/scm-repository-selection.md#qt-теги-топики-лейблы
