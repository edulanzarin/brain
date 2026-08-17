---
tags: [tipo/atomica, camada/principio, dev/backend, dev/frontend]
criado: 2026-08-17
---

# Estado vivo se empurra, não se pergunta

> Quem produz o estado avisa quando ele muda; a interface assina um canal, não fica perguntando de novo.

## A regra

Dado que muda sozinho (uma sessão rodando, um contador, um feed) tem um dono — o
processo que o produz. O caminho certo é o dono **empurrar** a mudança por um canal
(SSE, WebSocket, webhook, evento) e todo mundo que exibe **assinar** esse canal.
Perguntar de novo (polling) é exceção, e exceção precisa de motivo: não existe canal,
não existe IP público, a fonte é de terceiro e só oferece GET.

O sintoma de que o princípio foi violado é a **constelação de pollings**: cada tela
pergunta no seu próprio ritmo, os números discordam entre si por segundos, o que
ninguém perguntou fica velho até o F5, e o custo cresce por tela, não por mudança.

## Por que

Polling paga por pergunta, mesmo sem resposta nova; push paga por mudança. E com N
consumidores perguntando em ritmos diferentes, o estado exibido nunca é um só — cada
painel vive num instante diferente do mesmo sistema. Um canal único devolve uma
verdade única e instantânea.

Casos concretos: a área VIP do piwdex tinha 8 `setInterval` (2s a 10s) e três telas
que só atualizavam no F5 — virou um stream SSE único alimentado pelos eventos do
próprio robô ([[Um stream SSE substitui a constelação de pollings]]). O inverso
justificado existe e está anotado: [[Polling substitui webhook quando não há IP público]].
No frontend, a mesma espinha: [[Mundo imperativo e React se falam por eventos, não por referência]].

## Conexões
- Técnica: [[Um stream SSE substitui a constelação de pollings]]
- Exceção justificada: [[Polling substitui webhook quando não há IP público]]
- Irmã: [[Total ao vivo é o persistido fechado mais o em andamento ainda não gravado]]
- Mapa: [[Base]]
