# push

Команда `push` — [прокси-команда][PROXY-TERM] с git-глаголом `push`.
Префикс, хвост, отбор, порог, токен, итог и коды выхода —
[общий контракт][proxy]. Замысел —
[массовые локальные git-операции][mass-ops].

## Синтаксис

```text
githerd [глобальные опции] push [флаги префикса] [хвост git push]
```

Глобальные флаги — [глобальные параметры CLI][cli-index]. Команда не
вызывает API SCM: `upstream` и `token` не обязательны. Если они заданы,
в процесс `git` не передаются — [подключение к SCM][cli-scm].

## Примеры

```text
githerd push -A --force-with-lease
githerd push -- --all
```

Разбор флагов в этих примерах — [префикс и хвост][proxy-prefix].

## Трассировка вверх

- [Массовые локальные git-операции][mass-ops]
- [BG-5][bg-5]
- [ADR-26101702][adr-git] — `git` с машины пользователя
- [ADR-26110801][adr-layout] — компоненты Invocation и GitCliRunner

## Не описывать здесь

- Разбор префикса, запуск git и коды выхода — [общий контракт][proxy].
- Код в `application/`.

<!-- LINKS -->

[PROXY-TERM]: ../../../../glossary/terms/proxy-command.md
[proxy]: ../../platform/proxy-command.md
[proxy-prefix]: ../../platform/proxy-command.md#префикс-и-хвост
[mass-ops]: ../../../product-intent/capability/local/mass-local-git-operations.md
[bg-5]: ../../../product-intent/motivation/goals.md#bg-5
[cli-index]: ../index.md
[cli-scm]: ../index.md#подключение-к-scm
[adr-git]: ../../../../ADR/ADR-26101702-host-git-from-path.md
[adr-layout]: ../../../../ADR/ADR-26110801-go-package-layout.md
