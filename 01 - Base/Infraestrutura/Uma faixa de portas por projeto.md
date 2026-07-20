---
tags: [tipo/atomica, camada/principio, infra]
criado: 2026-07-20
---

# Uma faixa de portas por projeto

> Cada projeto tem sua faixa reservada. Escolher porta deixa de ser adivinhação e
> vira consulta a uma lista.

## A regra

O app fica na faixa `4xxx` (4001–4999) — a faixa das automações. **O banco espelha o
app trocando o `4` inicial por `5`**: app `4004` → banco `5004`, app `4011` → banco
`5011`. Os três últimos dígitos são os mesmos, então a porta identifica o projeto
sozinha e eu nunca preciso testar se está livre.

Mapa atual:

| Projeto | App | Banco |
|---|---|---|
| [[Cofre Digital]] | 4004 | 5004 |
| [[Navedesk]] | 4001 | 5401 |
| [[Questor Hub]] | 4022 | 5433 |
| [[Navecon Controller]] | 4088 | 5432 |

Projeto novo pega o próximo número livre e **registra aqui na hora** — a nota é a fonte
da verdade, não o `docker ps` da máquina que por acaso está ligada.

## O par são duas portas, não uma

"O projeto roda na 4004" quer dizer que **o app** atende em 4004. O banco do mesmo
projeto não atende na 4004 também — duas coisas não escutam a mesma porta do mesmo
host. Daí o par: `4004` app, `5004` banco, o mesmo número com o primeiro dígito trocado.

E o `5xxx` não briga com o Postgres "padrão": 5432 é a porta **dentro** do container, e
segue 5432 em todo projeto. O `5004` é só onde o host publica aquele container — ver
[[Porta interna é constante, porta externa é configuração]].

## Por que

O erro que isso evita não é o conflito de porta (esse o Docker grita na hora). É o
silencioso: dois projetos com o mesmo banco em 5432, eu abro o cliente SQL apontando
pro que achei que era, e rodo query no banco errado. Com faixa reservada, a porta
identifica o projeto sozinha.

Só o Cofre Digital segue o espelho `4xxx`→`5xxx` por enquanto. Os outros vieram antes
dele: Questor BI (5433) e Navecon Controller (5432) nunca tiveram sufixo nenhum, e
Navedesk (5401) ainda usa o formato antigo `54xx`. Ficam como estão — repontar porta de
projeto rodando não paga o risco; o espelho vale daqui pra frente.

## Conexões
- Irmã: [[Porta interna é constante, porta externa é configuração]] · [[O nome do projeto governa o nome dos recursos]]
- Mapa: [[Base]] · [[Infra]]
