---
tags: [tipo/atomica, camada/padrao, dev/backend]
criado: 2026-08-16
---

# Descobrir o shard por sondagem paralela com early-exit e cachear

> Quando um servico sharded te bota num no especifico mas **nao diz qual** (sem
> campo no token, sem endpoint de descoberta) e conectar no errado so devolve
> "wrong-shard", da pra achar o seu **abrindo todos em paralelo** e ficando com o
> primeiro que responder com dado. Depois **cacheia** — o normal vira 1 conexao.

## O contexto

Servicos realtime (WebSocket) as vezes distribuem contas por shard/no
(`ws1..wsN`) e assumem que o cliente ja sabe o seu — o bundle carrega essa config.
Reimplementando o cliente por fora (companion, bot, integracao), voce nao tem essa
config, e o shard nao aparece em lugar nenhum acessivel (nem no JWT, nem na REST).
Conectar no shard errado fecha com um code de erro (ex.: `4003 wrong-shard`).

## A solucao

O erro rapido e a chave: **shard errado rejeita quase instantaneo**, o certo aceita
e manda dado. Entao:

1. Abre `ws1..wsN` **todos em paralelo**.
2. **Early-exit**: resolve no primeiro socket que entregar um frame de dado valido;
   aborta/fecha os outros. Os errados ja se fecharam sozinhos com o code de rejeicao.
3. **Cacheia** o shard achado (no banco, junto do vinculo). A partir dai, tenta o
   cacheado com UMA conexao; so re-varre se ele passar a rejeitar (re-shard/migracao).

A varredura inteira custa ~o tempo de UM handshake (o certo responde junto com as
rejeicoes), nao N sequenciais. Poe um timeout de guarda pro caso de nenhum aceitar.

## Por que nao forca bruta ingenua

Varrer **sequencial** (tenta 1, espera fechar, tenta 2...) multiplica o timeout por N
e fica lento. O paralelo + early-exit colapsa isso porque as rejeicoes sao simultaneas
e o vencedor aparece na primeira leva. E sem o **cache** voce reabriria N conexoes a
cada request — rude com o servidor e desnecessario, ja que o shard e estavel.

## Conexoes
- Visto em: [[piwdex]] (WS de pokemons ativos, `ws<N>?token=`, cache em `game_links.shard`)
- Referencia concreta: [[Poke Idle World - endpoints publicos de dados]]
- Parente: [[Polling substitui webhook quando não há IP público]] (outro "arranjo" de
  realtime quando falta a via oficial)
- Mapa: [[Backend]]
