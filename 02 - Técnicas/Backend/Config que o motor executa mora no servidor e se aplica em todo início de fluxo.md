---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-18
---

# Config que o motor executa mora no servidor e se aplica em todo início de fluxo

> Config de automação não viaja no request de start vinda do localStorage: o cliente
> salva no servidor, e o SERVIDOR aplica a config salva em todo caminho que inicia o fluxo.

## O problema

A config (travas de venda do robô) vivia no localStorage e era anexada ao POST de start
por UM componente da UI. Qualquer outro caminho que iniciasse o mesmo fluxo (botão do
painel, boot do container, retomada) largava o motor SEM a config — com a tela de
configurações dizendo "Ligado". Pior: parar o fluxo apagava a config persistida, então
até o caminho certo esquecia a escolha do usuário.

Assinatura do bug: o recurso funciona quando ativado por um lugar e falha ativado por
outro; e some depois de parar/religar.

## A solução

- A config é **salva no servidor** (linha por usuário, ex.: `robot_sessions.poke_sell_cfg`)
  com um campo `on` embutido — desligar preserva as travas, só vira `on:false`.
- **Todo** handler que inicia o fluxo (start manual, auto, plano, connect, boot) carrega
  a config salva e aplica. O cliente não transporta config; no máximo edita e salva.
- Parar o fluxo não apaga config: só o interruptor explícito do usuário mexe nela.

## O que mais vale lembrar

- O corolário de UI: o toggle da config lê/grava o servidor (GET/POST), nunca uma chave
  local — senão dois lugares mostram verdades diferentes.
- Aceitar config explícita no body ainda é útil (compat/override), mas é exceção, não o
  mecanismo.

## Conexões
- Princípio: [[Estado mutável se lê da fonte no uso, não de cópia guardada]] · [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[Guarde a intenção e o processo se reconstrói dela]] · [[Estado desejado persistido religa o robô depois do restart]]
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
