# remote

Команда `remote` — [прокси-команда][PROXY-TERM] с git-глаголом `remote`.
Префикс, хвост, отбор, порог, токен, итог и коды выхода —
[общий контракт][proxy]. Замысел —
[массовые локальные git-операции][mass-ops].

Слова `add`, `set-url`, `remove` и другие подкоманды git входят в хвост.
Отдельной командой Git Herder они не становятся.

## Синтаксис

```text
githerd [глобальные опции] remote [флаги префикса] [хвост git remote]
```

Глобальные флаги — [глобальные параметры CLI][cli-index]. Команда не
вызывает API SCM: `upstream` и `token` не обязательны. Если они заданы,
в процесс `git` не передаются — [подключение к SCM][cli-scm].

## Примеры

```text
githerd remote -v
githerd remote -- -v
githerd remote add -t main origin git@github.com:org/billing.git
githerd remote set-url origin --push git@github.com:org/billing.git
githerd remote add origin git@gitlab.example.com:group/{path}.git
githerd remote set-url origin git@github.com:myorg/{name}.git
```

Первая печатает версию. Вторая запускает `git remote -v`. Разбор
`-v` — [префикс и хвост][proxy-prefix]. Последние две раскрывают
`{path}` и `{name}` по [подстановке в хвосте][proxy-subst]. Имя
команды называет `git remote`, а не [ремот][REMOTE-TERM].

## Трассировка вверх

- [Массовые локальные git-операции][mass-ops]
- [BG-7][bg-7]
- [ADR-26101702][adr-git] — `git` с машины пользователя
- [ADR-26110801][adr-layout] — компоненты Invocation и GitCliRunner

## Не описывать здесь

- Разбор префикса, подстановка в хвосте, запуск git и коды выхода —
  [общий контракт][proxy].
- Код в `application/`.

<!-- LINKS -->

[PROXY-TERM]: ../../../../glossary/terms/proxy-command.md
[REMOTE-TERM]: ../../../../glossary/terms/remote.md
[proxy]: ../../platform/proxy-command.md
[proxy-prefix]: ../../platform/proxy-command.md#префикс-и-хвост
[proxy-subst]: ../../platform/proxy-command.md#подстановка-в-хвосте
[mass-ops]: ../../../product-intent/capability/local/mass-local-git-operations.md
[bg-7]: ../../../product-intent/motivation/goals.md#bg-7
[cli-index]: ../index.md
[cli-scm]: ../index.md#подключение-к-scm
[adr-git]: ../../../../ADR/ADR-26101702-host-git-from-path.md
[adr-layout]: ../../../../ADR/ADR-26110801-go-package-layout.md
