# claude-plugins

Marketplace pessoal de plugins do Claude Code, um plugin por domínio.

| Plugin | Comandos | Hook |
|---|---|---|
| `git` | `/git:commit`, `/git:chamado <n>`, `/git:main` | valida commit (semântico, body, sem co-author, sem conjunção), push e main; audita commits na main em `~/.claude/git-main-audit.log` |
| `gitlab` | `/gitlab:mr [@revisor\|@auto] [draft]` | valida título e descrição do MR, bloqueia commit pela API (`add_commit`) |

Os comandos se combinam nos argumentos: `/git:commit /git:chamado 19738 /gitlab:mr @fulano`.

## Instalar

```bash
claude plugin marketplace add ~/Development/claude-plugins
claude plugin install git@kauan
claude plugin install gitlab@kauan
```

O `gitlab` depende do MCP `gitlab` configurado no usuário.

## Alterar

O Claude Code copia o plugin para o cache por versão. Depois de editar, suba o `version` em `plugins/<nome>/.claude-plugin/plugin.json` e rode `claude plugin update <nome>@kauan`. Para testar sem instalar: `claude --plugin-dir plugins/<nome>`.

## Testar

```bash
node plugins/git/hooks/guard.test.js
node plugins/gitlab/hooks/guard.test.js
```
