---
tags: [tipo/atomica, camada/principio, dev, armadilha]
criado: 2026-07-18
---

# Verificar no build de produção, não só em dev

> Alguns bugs só aparecem em `build`/`start`; validar a feature no build de produção, não apenas no dev server.

## O quê

Dev servers ligam otimizações, cache, tree-shaking e comportamentos de roteamento diferentes da produção. Uma feature pode passar 100% em `next dev` e falhar calada em `next build && next start`. Aconteceu no [[Questor Hub]]: ver [[router.replace do Next falha no build de produção]].

Prática: ao terminar algo não-trivial, rodar o build de produção real e exercitar o fluxo (nem que seja com um screenshot via browser headless), não confiar só no dev.

## Por que importa

O custo de descobrir isso em produção (com usuário) é muito maior que rodar um build a mais. É barato e pega uma classe inteira de bugs invisíveis em dev.

## Conexões
- Relacionado: [[router.replace do Next falha no build de produção]]
- Mapa: [[Base]]
