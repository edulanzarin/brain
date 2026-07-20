---
tags: [tipo/pensamento]
criado: 2026-07-18
---

# Manter o tooling enxuto e o conhecimento no cérebro

> Prefiro deixar as ferramentas (config, memória do assistente) enxutas e guardar
> o conhecimento pesado no cérebro, consultado só quando precisa.

## O pensamento

O Eduardo prefere não poluir a raiz de configuração do assistente (`~/.claude`)
com detalhe de projeto. A intuição: contexto que é carregado em TODA sessão custa
atenção/token à toa; conhecimento que fica no cérebro e é lido **sob demanda** só
pesa quando é relevante. Então a memória do assistente vira um **ponteiro** curto
("existe o projeto X, o detalhe está no cérebro Y") e o conteúdo mora nas notas.

## Por que importa

Escala melhor: com muitos projetos, uma config enxuta continua barata, e cada
cérebro ([[Questor Hub]] pessoal, cérebro técnico do banco) cresce sem inchar o
default. É a mesma lógica de [[Agregar antes de juntar em tabelas gigantes no Postgres]]:
não carregar o que você não vai usar naquele momento.

## Nuance

Vale corrigir uma crença comum: apagar histórico de sessão não economiza token —
esse histórico não volta pro contexto. O que economiza é manter enxuto o que É
recarregado (as instruções globais e a memória-ponteiro).

## Conexões
- Visto em: [[Questor Hub]]
- Mapa: [[Pensamentos]]
