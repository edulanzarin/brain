---
tags: [tipo/atomica, camada/principio, infra]
criado: 2026-07-20
---

# Uma faixa de portas por projeto

> Cada projeto tem sua faixa reservada. Escolher porta deixa de ser adivinhação e
> vira consulta a uma lista.

## A regra

Faixa `40xx` pro app, `54xx` pro banco, com o mesmo final ligando os dois. Projeto que
pega `4011` pega `5411`. Assim, olhando a porta eu sei de quem é, e nunca preciso testar
se está livre.

Mapa atual:

| Projeto | App | Banco |
|---|---|---|
| [[Navedesk]] | 4001 | 5401 |
| [[Cofre Digital]] | 4004 | 5404 |
| [[Questor BI]] | 4022 | 5433 |
| [[Navecon Controller]] | 4088 | 5432 |

Projeto novo pega o próximo número livre e **registra aqui na hora** — a nota é a fonte
da verdade, não o `docker ps` da máquina que por acaso está ligada.

## Por que

O erro que isso evita não é o conflito de porta (esse o Docker grita na hora). É o
silencioso: dois projetos com o mesmo banco em 5432, eu abro o cliente SQL apontando
pro que achei que era, e rodo query no banco errado. Com faixa reservada, a porta
identifica o projeto sozinha.

Questor BI e Navecon Controller ficaram fora do padrão de sufixo (5433, 5432) porque
nasceram antes da regra. Ficam como estão — renomear porta de projeto rodando não paga
o risco; a regra vale daqui pra frente.

## Conexões
- Irmã: [[Porta interna é constante, porta externa é configuração]] · [[O nome do projeto governa o nome dos recursos]]
- Mapa: [[Base]] · [[Infra]]
