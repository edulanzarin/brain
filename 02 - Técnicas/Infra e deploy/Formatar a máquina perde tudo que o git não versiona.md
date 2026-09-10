---
tags: [tipo/atomica, camada/padrao, infra]
criado: 2026-09-10
---

# Formatar a máquina perde tudo que o git não versiona

> Antes de formatar, a pergunta não é "quais projetos eu tenho", é "o que de cada
> projeto está fora do git". A resposta é sempre maior do que parece.

## O problema

A sensação é de que está tudo no GitHub. Na varredura antes de uma formatação apareceu o
contrário: metade dos projetos sem remote, cinco com trabalho não commitado, um com commits
sem push, 27 `.env`, 15 bancos de dev só em volume Docker e a configuração do Claude
(skills, memórias, CLAUDE.md global) solta em `~/.claude`. Nada disso incomoda no dia a
dia. Tudo some na formatação.

## A solução

Varrer por categoria, não por projeto. Cada categoria tem um comando que responde para
todas as pastas de uma vez:

| O que fica fora | Como achar | Para onde vai |
|---|---|---|
| Repositório sem remote | `git remote -v` vazio | repositório privado novo |
| Commit sem push | `git rev-list --count @{u}..HEAD` | push |
| Trabalho não commitado | `git status --porcelain` | commit numa branch `wip/*`, sem revisar |
| Pasta sem git | não tem `.git` | `git init` com o build no `.gitignore` |
| Segredo | `.env*` que `git ls-files --error-unmatch` não conhece | cofre |
| Ignorado que não se regenera (upload, asset bruto) | `git status --ignored`, tirando `node_modules`, `.next` e cache | cofre |
| Banco de dev | containers `*-db` em `docker ps -a` | `pg_dumpall` por container, gzip, cofre |
| Config de ferramenta (`~/.claude`, shell, editor) | olhar a home | repositório `dotfiles` privado |

Duas rotas, separadas pelo que pode vazar. Código e configuração vão para o GitHub
privado. Segredo e dado de pessoa (`.env`, dump, PDF de RH) vão para um cofre offline e
nunca entram no git.

A volta é um `RESTAURAR.md` escrito para o Claude executar na máquina limpa: etapas em
ordem, comandos idempotentes, e marcado onde precisa do humano (senha, navegador, pendrive).

## O que mais vale lembrar

- **Nome de repositório colide.** A pasta local e o repositório de mesmo nome no GitHub
  podem ter históricos sem relação (reescrita, versão antiga). `git ls-remote` pela chave
  SSH diz se o nome existe. Se existe, `fetch` para um namespace temporário e
  `merge-base --is-ancestor` dizem se o local já está lá. Nunca force-push por cima.
- **Memória do Claude é endereçada pelo caminho.** A pasta em `~/.claude/projects/` é o
  caminho absoluto com `/` trocado por `-`. Projeto que volta em outra pasta perde a
  memória, por isso a volta clona cada um na mesma pasta de antes.
- **Chave SSH.** Se o `known_hosts` só tem github.com, gerar chave nova é mais seguro que
  carregar a antiga num pendrive. Se tiver servidor, a chave é acesso a produção e vai
  para o cofre.
- **Cache grande engana.** 900 MB num projeto eram `.pnpm-store`. Medir sem
  `node_modules`, `.next` e cache antes de concluir que há dado a salvar.
- **Skill de terceiro instalada solta** (sem git, de várias origens) não se baixa de novo
  com confiança. Copiar a pasta inteira.

## Conexões
- Princípio: [[Configuração vem do ambiente, não do código]]
- Irmã: [[Volume de dev sobrevive entre versões do projeto e traz schema velho]]
- Mapa: [[Infra]]
