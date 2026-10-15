# kamoca

## Sumário

- [Instalação](#instalação)
- [Contexto](#contexto)
- [Catálogo](#catálogo)
- [Como as regras são garantidas](#como-as-regras-são-garantidas)
- [Estrutura do repo](#estrutura-do-repo)
- [Documentação](#documentação)

## Instalação

```bash
git clone https://github.com/KauanCalheiro/claude-plugins.git ~/Development/claude-plugins
claude plugin marketplace add ~/Development/claude-plugins
claude plugin install git@kamoca
claude plugin install craft@kamoca
claude plugin install docker@kamoca
```

Requer Node 22.18+ (os hooks são `.ts` executados direto pelo Node). Os comandos aparecem na próxima sessão. O repo é privado: o clone usa suas credenciais do GitHub (`gh auth login`).

## Contexto

Marketplace pessoal de plugins do [Claude Code](https://code.claude.com), organizado por domínio. Cada plugin junta as skills (os comandos `/dominio:acao`) e os hooks que **garantem** as regras daquele domínio, em vez de depender do modelo lembrar delas.

As skills de commit nasceram soltas em `~/.claude`, sem versão. Este repo as reúne num marketplace (`kamoca`) para que:

- tudo fique versionado e revisado por PR/MR, como qualquer código;
- cada domínio (git, craft, docker) evolua e seja instalado de forma independente;
- as regras de histórico (commit semântico, body explicando o porquê, nada direto na main) e as boas práticas (ATDD, refactor) valham em **todos** os projetos, do trabalho e pessoais.

Aqui só entra o que é geral. O que é de um lugar só (a integração com o GitLab do trabalho, por exemplo) fica num marketplace próprio e se combina com estes pelos argumentos dos comandos.

## Catálogo

| Plugin | Comando | O que faz |
|---|---|---|
| [`git`](plugins/git/README.md) | `/git:commit` | Commits semânticos atômicos com body, em branch dedicada, e push |
| | `/git:main` | Autoriza commit e push na main, com justificativa e auditoria |
| | `/git:sync` | Depois do merge, volta para a main atualizada e apaga a branch mergeada |
| | `/git:split` | Quebra uma branch grande em branches encadeadas, uma por assunto |
| | `/git:undo` | Desfaz commit, amend, rebase ou reset pelo reflog, com backup |
| [`craft`](plugins/craft/README.md) | `/craft:craft` | Regra raiz (menos código é melhor do que mais código) e triagem de SOLID, KISS, YAGNI, DRY e Clean Code; carrega só a skill de cada princípio com sinal. O hook dispara isso sozinho quando a suíte fica verde |
| | `/craft:setup` | Prepara um projeto novo: teste, linter com limites de tamanho e complexidade, scripts `test` e `lint` |
| | `/craft:atdd` | Conduz o fluxo ATDD de 7 passos, da especificação Given/When/Then ao verde final |
| | (automático) | Hook carrega em toda sessão a skill `craft:ste`: respostas e textos no padrão ASD-STE100 |
| [`docker`](plugins/docker/README.md) | `/docker:docker` | Entrada única: roteia para create ou tune, com sub-skills privadas compartilhadas entre os dois |
| | `/docker:create` | Imagens na versão mais nova, nginx com HTTP/2 e HTTP/3; dev com hot reload e UID/GID do host, prod enxuto em `./deploy` com `.env` próprio, defaults, volumes em pasta local, nomes padronizados |
| | `/docker:tune` | Põe o Docker existente nas regras e otimiza com medição, uma mudança por vez |

Os comandos se combinam nos argumentos, inclusive com os de outros marketplaces:

```
/git:commit /<plugin>:<comando> @fulano
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
.claude-plugin/marketplace.json     lista os plugins do marketplace "kamoca"
plugins/
├── git/
│   ├── .claude-plugin/plugin.json
│   ├── skills/{commit,main,sync,split,undo}/SKILL.md
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
├── craft/
│   ├── .claude-plugin/plugin.json
│   ├── skills/craft/SKILL.md      triagem dos princípios
│   ├── skills/atdd/SKILL.md
│   ├── skills/<princípio>/SKILL.md  um por princípio
│   └── hooks/                   mesma estrutura do git
└── docker/
    ├── .claude-plugin/plugin.json
    ├── skills/docker/SKILL.md     hub: roteia para create ou tune
    ├── skills/{create,tune}/SKILL.md
    ├── skills/<assunto>/SKILL.md  sub-skills privadas (user-invocable: false)
    └── hooks/                   mesma estrutura do git
```

## Documentação

| Documento | Conteúdo |
|---|---|
| [USAGE.md](USAGE.md) | Atualizar, testar, criar regra nova, transformar caso real em teste, combinar comandos, solução de problemas |
| [plugins/git/README.md](plugins/git/README.md) | Catálogo e regras do plugin `git` |
| [plugins/git/USAGE.md](plugins/git/USAGE.md) | Casos de uso do `git`, com exemplos de mensagens aceitas e negadas |
| [plugins/craft/README.md](plugins/craft/README.md) | Comandos, skills por princípio, hook de refactor e executores de teste reconhecidos |
| [plugins/craft/USAGE.md](plugins/craft/USAGE.md) | ATDD em feature e bug, refactor automático e sob demanda |
| [plugins/docker/README.md](plugins/docker/README.md) | Comandos e regras do plugin `docker` |
| [plugins/docker/USAGE.md](plugins/docker/USAGE.md) | Criar, subir dev e prod, ajustar com medição, negações dos hooks |
