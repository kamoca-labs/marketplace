# claude-plugins

## Sumário

- [Instalação](#instalação)
- [Contexto](#contexto)
- [Catálogo](#catálogo)
- [Como as regras são garantidas](#como-as-regras-são-garantidas)
- [Estrutura do repo](#estrutura-do-repo)
- [Documentação](#documentação)

## Instalação

```bash
git clone https://github.com/kamoca-labs/marketplace.git ~/Development/claude-plugins
claude plugin marketplace add ~/Development/claude-plugins
claude plugin install git@kauan
claude plugin install gitlab@kauan
claude plugin install practices@kauan
```

Requer Node 22.18+ (os hooks são `.ts` executados direto pelo Node). Os comandos aparecem na próxima sessão. O `gitlab` já traz o MCP do GitLab: só falta autenticar com `/mcp` (veja os [pré-requisitos](plugins/gitlab/README.md#pré-requisitos)).

## Contexto

Marketplace pessoal de plugins do [Claude Code](https://code.claude.com), organizado por domínio. Cada plugin junta as skills (os comandos `/dominio:acao`) e os hooks que **garantem** as regras daquele domínio, em vez de depender do modelo lembrar delas.

As skills de commit e merge request nasceram soltas em `~/.claude`, sem versão. Este repo as reúne num marketplace (`kauan`) para que:

- tudo fique versionado e revisado por MR, como qualquer código;
- cada domínio (git, gitlab, practices) evolua e seja instalado de forma independente;
- as regras de histórico (commit semântico, body explicando o porquê, nada direto na main) e as boas práticas (ATDD, refactor) valham em **todos** os projetos, não só em um.

## Catálogo

| Plugin | Comando | O que faz |
|---|---|---|
| [`git`](plugins/git/README.md) | `/git:commit` | Commits semânticos atômicos com body, em branch dedicada, e push |
| | `/git:chamado <n>` | Usa o número do chamado como escopo; sozinho, cria a branch do chamado |
| | `/git:main` | Autoriza commit e push na main, com justificativa e auditoria |
| [`gitlab`](plugins/gitlab/README.md) | `/gitlab:mr` | Abre ou atualiza o MR via MCP, com descrição obrigatória e revisores |
| [`practices`](plugins/practices/README.md) | `/practices:atdd` | Conduz o fluxo ATDD de 7 passos, da especificação Given/When/Then ao verde final |
| | `/practices:refactor` | Aplica SOLID, KISS, YAGNI, DRY e Clean Code; o hook injeta isso sozinho quando a suíte fica verde |

Os comandos se combinam nos argumentos:

```
/git:commit /git:chamado 19738 /gitlab:mr @fulano
```

## Como as regras são garantidas

| Camada | Papel |
|---|---|
| **Skill** (`skills/*/SKILL.md`) | Diz ao Claude **como** fazer: passos, formato, modelos |
| **Hook** (`hooks/rules/<regra>.ts`) | **Nega** a chamada que viola a regra, antes de executar |

Cada regra é um arquivo, com hook próprio no `hooks.json` e teste próprio em `hooks/tests/<regra>.test.ts`. Quando uma chamada viola várias regras, todas aparecem de uma vez, cada uma com seu prefixo (`[git:semantic-title]`, `[git:commit-body]`).

O hook usa o campo `if:` do Claude Code (`Bash(git commit *)`, `Bash(git push *)`) para só rodar quando interessa. Como esse filtro é best-effort, cada regra confere de novo se o comando é mesmo um commit ou um push.

Autorizações como `/git:main` são lidas do **último prompt digitado por você** no transcript da sessão. O Claude não consegue se autorizar sozinho.

Todo hook é testado em três níveis:

| Nível | Arquivo | Garante |
|---|---|---|
| Regra | `tests/<regra>.test.ts` | a regra isolada nega e libera o que deve |
| Caso de uso | `tests/cases.json` | o cenário real (comando, branch, prompt) passa por **todos** os hooks e só as regras esperadas negam |
| Estrutura | `tests/structure.test.ts` | nenhuma regra fica sem hook dedicado ou sem teste |

Cada negação em uso real é gravada em `~/.claude/hook-cases.jsonl` já no formato do `cases.json`. Quando um hook nega o que não devia, ou deixa passar o que devia negar, o caso entra no `cases.json` e vira teste. Veja [transformar caso real em teste](USAGE.md#transformar-caso-real-em-teste).

## Estrutura do repo

```
.claude-plugin/marketplace.json     lista os plugins do marketplace "kauan"
plugins/
├── git/
│   ├── .claude-plugin/plugin.json
│   ├── skills/{commit,chamado,main}/SKILL.md
│   └── hooks/
│       ├── hooks.json           um hook por regra
│       ├── lib.ts               leitura do payload, mensagem, prompt
│       ├── rules/<regra>.ts     uma regra por arquivo
│       └── tests/
│           ├── <regra>.test.ts  um teste por regra
│           ├── cases.json       casos de uso de ponta a ponta
│           ├── cases.test.ts    roda o cases.json contra o hooks.json
│           ├── capture.test.ts  negação real vira caso
│           └── structure.test.ts
├── gitlab/
│   ├── .claude-plugin/plugin.json
│   ├── .mcp.json                servidor MCP do GitLab
│   ├── skills/mr/SKILL.md
│   └── hooks/                   mesma estrutura do git
└── practices/
    ├── .claude-plugin/plugin.json
    ├── skills/{atdd,refactor}/SKILL.md
    └── hooks/                   mesma estrutura do git
```

## Documentação

| Documento | Conteúdo |
|---|---|
| [USAGE.md](USAGE.md) | Atualizar, testar, criar regra nova, transformar caso real em teste, combinar comandos, solução de problemas |
| [plugins/git/README.md](plugins/git/README.md) | Catálogo e regras do plugin `git` |
| [plugins/git/USAGE.md](plugins/git/USAGE.md) | Casos de uso do `git`, com exemplos de mensagens aceitas e negadas |
| [plugins/gitlab/README.md](plugins/gitlab/README.md) | Catálogo, regras e pré-requisitos (MCP) do plugin `gitlab` |
| [plugins/gitlab/USAGE.md](plugins/gitlab/USAGE.md) | Casos de uso do `gitlab`: revisores, draft, chamado, atualização de MR |
| [plugins/practices/README.md](plugins/practices/README.md) | Comandos, hook de refactor e executores de teste reconhecidos |
| [plugins/practices/USAGE.md](plugins/practices/USAGE.md) | ATDD em feature e bug, refactor automático e sob demanda |
