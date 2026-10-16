# guard-shell — shell не обходит защиту

Событие: **PreToolUse**, matcher `Bash` (в Codex сюда же попадает `exec_command`).
Команда: `wk hook guard-shell`.
Действие: команда меняет проверки или их конфигурацию → **код 2**, одна строка причины.

Хуки записи видят только `Edit|Write|MultiEdit|NotebookEdit`. Без этого хука
`rm .claude/settings.json` или `sed -i` по манифесту из shell снимали защиту целиком:
Claude Code подхватывает изменение настроек на лету.

## Что блокируется

| условие | пример |
|---|---|
| упомянут защищённый путь из `guard-checker` | `cat .wiki-project.yaml`, `cd .claude && rm settings.json` |
| запись в `.claude`, `.codex`, `_meta` или файл блока политик | `rm -rf .codex`, `printf x >> AGENTS.md` |
| перезакрепление toolchain или повторная инициализация | `nix flake update`, `nix flake lock`, `wk init` |
| `apply_patch` внутри команды | проверяется так же, как `guard-paths` и `guard-block` |

«Запись» — перенаправление `>`, `tee`, `rm`/`mv`/`cp`/`ln`/`dd`/`truncate`/`touch`,
`sed -i`/`perl -i`, `git checkout|restore|reset|stash|apply|clean`, интерпретаторы
(`python`, `node`, `bash`, `awk`, `eval`, `xargs`). Чтение блока политик (`grep`, `cat`)
разрешено; защищённые файлы агент читает инструментом Read.

## Границы

Это растяжка, а не парсер shell: `eval "$(base64 -d …)"`, переменные и glob её обходят.
Ложное срабатывание стоит агенту одного вызова Read, пропуск — гейта.
Гарантию даёт CI с `wk check --strict` и `wk`, закреплённым вне репозитория проекта.
