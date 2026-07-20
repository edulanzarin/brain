---
tags: [tipo/atomica, camada/principio, cerebro]
criado: 2026-07-20
---

# Camadas do conhecimento - princípio, padrão, aplicação

> Todo conhecimento do cérebro mora em uma de três camadas, e o link sempre sobe:
> aplicação aponta pro padrão, padrão aponta pro princípio.

## As três camadas

**Princípio** (`camada/principio`, pasta `Base/`) — a regra atemporal, independente
de stack e de projeto. Responde *por quê*. Sobrevive à troca de framework, de
linguagem e de emprego. Ex: [[Token semântico em vez de valor literal]].

**Padrão** (`camada/padrao`, pastas `Design/`, `Stack/`) — a aplicação concreta do
princípio numa tecnologia. Responde *como*. Morre quando a tecnologia morre. Ex:
[[Sistema de cores e tema do dashboard]] (o princípio acima, em CSS vars + Tailwind).

**Aplicação** (`tipo/projeto`, pasta `01 - Projetos/`) — onde os padrões foram usados
de verdade, com as decisões e o estado daquele sistema. Responde *onde*.

## Por que separar

Sem camada, tudo vira "uma nota sobre X" e o cérebro só cresce em largura. Com camada,
ele cresce em **profundidade**: quando aparece um problema novo, dá pra perguntar "qual
princípio já cobre isso?" antes de inventar solução. Quase sempre já existe um, e o
trabalho vira escrever só o padrão novo abaixo dele.

É também o que faz o cérebro escalar: princípios são poucos e estáveis, padrões são
muitos e descartáveis, projetos são temporários. Cada camada tem um ritmo de mudança
diferente e não deve arrastar as outras.

## A regra do link

Link **sobe** e é obrigatório: aplicação → padrão → princípio. Link **desce** é opcional
e enxuto (um princípio cita um ou dois padrões de exemplo, não todos). Nunca desça até
a camada de projeto — ver [[Conhecimento pertence à base, não ao projeto]].

## Conexões
- Regra irmã: [[Conhecimento pertence à base, não ao projeto]]
- Mapa: [[Base]]
