# Маппинг `qt` (GitHub)

Смысл флага — [hub § qt][scm-repository-selection-qt]. Как отфильтровать по тегам на
клиенте — [enumeration/qt.md](../repository-enumeration/qt.md).

## Когда есть смысл

После резолва задан хотя бы один `--qt`. Нет `qt` — этот маппинг не нужен.

## Запрос list

По [index § сборка q](index.md#сборка-q): **`qt` в `q` не пишем**.

Теги на карточке: поле `labels` из `topics` в ответе Search
([base-enumeration § карточка][base-enumeration-card]). Отдельного HTTP за labels
**нет** (на GitLab labels догружают в mapping `qt`).

## Не в этом файле

- Правила сравнения с `labels` — [enumeration/qt.md](../repository-enumeration/qt.md).

<!-- LINKS -->
[scm-repository-selection-qt]: ../../../platform/scm-repository-selection.md#qt-теги
[base-enumeration-card]: ../base-enumeration.md#карточка-репозитория
