# Uso do marketplace

Como manter, testar e combinar os plugins. Cada plugin vive no repo dele em `kamoca-labs/<plugin>`, e os caminhos abaixo valem a partir da raiz desse repo. A instalação está no [README](README.md#instalação). Os casos de uso de cada comando ficam no USAGE do plugin.

## Sumário

- [Atualizar depois de editar](#atualizar-depois-de-editar)
- [Testar sem instalar](#testar-sem-instalar)
- [Rodar os testes dos hooks](#rodar-os-testes-dos-hooks)
- [Criar uma regra nova](#criar-uma-regra-nova)
- [Transformar caso real em teste](#transformar-caso-real-em-teste)
- [Combinar comandos](#combinar-comandos)
- [Fluxo do dia a dia](#fluxo-do-dia-a-dia)
- [Solução de problemas](#solução-de-problemas)
- [Casos de uso por plugin](#casos-de-uso-por-plugin)

## Atualizar depois de editar

O Claude Code copia cada plugin para `~/.claude/plugins/cache/` **por versão**. Editar o repo não muda nada até você atualizar:

1. Suba o `version` em `.claude-plugin/plugin.json` do repo do plugin (ex.: `0.1.0` para `0.2.0`).
2. Rode `claude plugin marketplace update kamoca`.
3. Rode `claude plugin update <nome>@kamoca`.
4. Abra uma sessão nova.

## Testar sem instalar

Para experimentar uma mudança antes de subir a versão:

```bash
claude --plugin-dir ~/Developer/kamoca-labs/git
```

## Rodar os testes dos hooks

Tudo com `node:test`, sem dependência:

```bash
node --test 'hooks/tests/*.test.ts'                       # todos
node --test hooks/tests/commit-body.test.ts              # uma regra
node --test hooks/tests/cases.test.ts                    # casos de uso
```

Os três níveis (regra, caso de uso, estrutura) estão descritos no [README](README.md#como-as-regras-são-garantidas). O teste de estrutura falha se uma regra ficar sem hook no `hooks.json` ou sem `<regra>.test.ts`.

## Criar uma regra nova

1. **Teste primeiro** em `hooks/tests/<regra>.test.ts`, usando o `helpers.ts` da pasta, e confirme que ele falha.
2. **A regra** em `hooks/rules/<regra>.ts`: lê o payload com `readInput()`, nega com `deny('<regra>', 'motivo')` e termina sem fazer nada quando não se aplica.
3. **O hook** no `hooks/hooks.json`: uma entrada própria, com `if:` (Bash) ou `matcher` (MCP) e `statusMessage` `<plugin>:<regra>: ...`.
4. **Os casos de uso** no `hooks/tests/cases.json`: pelo menos um cenário que a regra nega e um que ela libera. Ajuste o `deny` dos casos existentes que a regra nova afeta.
5. **A documentação**: uma linha na tabela de regras do README do plugin e o caso de negação no USAGE.
6. Suba o `version` e [atualize](#atualizar-depois-de-editar).

Esqueleto de uma regra:

```ts
import { commitMessage, deny, readInput } from '../lib.ts'

const message = commitMessage(readInput())

if (message && /wip/i.test(message.subject)) {
  deny('no-wip', 'Commit WIP não entra no histórico.')
}
```

## Transformar caso real em teste

Cada negação feita pelos hooks em uso real vira uma linha em `~/.claude/hook-cases.jsonl`, já no formato do `cases.json`:

```json
{"plugin":"git","description":"Título fora do padrão semântico ...","origin":"real","branch":"feat/x","prompt":"/git:commit","tool_name":"Bash","tool_input":{"command":"..."},"deny":["semantic-title"]}
```

Ver as últimas negações:

```bash
tail -n 5 ~/.claude/hook-cases.jsonl | node -e 'require("readline").createInterface({input:process.stdin}).on("line",l=>{const c=JSON.parse(l);console.log(c.plugin, c.deny, c.description)})'
```

| Situação | O que fazer |
|---|---|
| **Negou certo** | nada; se o cenário for novo, vale guardar como regressão |
| **Falso positivo** (negou o que devia passar) | copie a linha para `hooks/tests/cases.json`, troque `deny` para `[]`, ajuste `origin` para `real <data>` e corrija a regra até o caso passar |
| **Falso negativo** (deixou passar o que devia negar) | não há linha capturada; escreva o caso à mão no `cases.json` com o `deny` esperado e corrija a regra |

O campo `plugin` só serve para saber em qual `cases.json` colar; o teste ignora. No `craft`, que injeta contexto em vez de negar, a linha traz `inject` no lugar de `deny` e o fluxo é o mesmo. O primeiro caso real do repo é o falso positivo do `$VAR` (`origin: "real 2026-10-06"` no `cases.json` do `git`).

## Combinar comandos

Só o primeiro `/comando` da linha é invocado. Os outros viram argumentos, e a skill invocada repassa cada um ao comando correspondente.

| Você digita | O que acontece |
|---|---|
| `/git:commit` | commita e faz push na branch atual (cria uma branch se estiver na main) |
| `/git:commit /<plugin>:<comando> @fulano` | commita, faz push e invoca o comando do outro plugin com o resto dos argumentos |
| `/git:commit /git:main` | commit direto na main, com justificativa e auditoria |
| `/git:split /<plugin>:<comando>` | quebra a branch e invoca o comando para cada parte, na ordem |

## Fluxo do dia a dia

```
/craft:atdd recusar anexo acima de 10 MB      especificação, teste vermelho, verde e refactor
/git:commit                                    cria a branch, commita e faz push
... revisão e merge ...
/git:sync                                      volta para a main atualizada e apaga a branch
```

## Solução de problemas

| Sintoma | Causa | Solução |
|---|---|---|
| `/git:commit` não aparece | plugin instalado na sessão atual | abra uma sessão nova |
| Mudei o repo e nada mudou | o plugin roda da cópia em cache | [atualize a versão](#atualizar-depois-de-editar) |
| `[git:<regra>] ...` negou o commit | a mensagem viola uma regra | veja [mensagens negadas](https://github.com/kamoca-labs/git/blob/main/USAGE.md#quando-o-hook-nega) |
| Hook negou algo que devia passar | falso positivo da regra | [transforme o caso em teste](#transformar-caso-real-em-teste) e corrija |
| `Marketplace name impersonates...` | `claude-plugins` é um nome reservado | o marketplace se chama `kamoca`; o repo é `kamoca-labs/marketplace` |
| Commit negado por outro hook (`[commit-hook]`) | hook do projeto aberto na sessão, não deste repo | rode o comando você mesmo com `!` no prompt |

## Casos de uso por plugin

- [git/USAGE.md](https://github.com/kamoca-labs/git/blob/main/USAGE.md)
- [craft/USAGE.md](https://github.com/kamoca-labs/craft/blob/main/USAGE.md)
- [docker/USAGE.md](https://github.com/kamoca-labs/docker/blob/main/USAGE.md)
