---
tags: [tipo/atomica, camada/padrao, dev/backend]
criado: 2026-08-18
---

# O bundle público do cliente entrega o contrato da API sem documentação

> Quando a API de um sistema externo não tem doc e o HAR não capturou a ação, o JS
> público do próprio cliente web contém as URLs e os payloads exatos — é só baixar os
> chunks e grepar.

## O problema

Implementar uma AÇÃO contra API de terceiro (ex.: comprar no mercado do jogo) sem
documentação. Chutar POST em endpoint adivinhado é inaceitável quando a ação gasta
dinheiro/é irreversível; esperar o usuário capturar um HAR da ação trava o trabalho.

## A solução

O cliente web oficial é código público que FAZ essas chamadas:

1. Baixar os chunks JS da página (`curl` no HTML, extrair `/_next/static/chunks/*.js`;
   os lazy aparecem referenciados dentro dos primeiros — baixar a segunda leva).
2. `grep -oE '"/api/[a-z/...]+"'` lista todos os endpoints; grepar o contexto da
   chamada revela o payload literal (ex.: `{action:"buy-stack",kind,refId,price,
   currency,quantity,ids}`).
3. O payload extraído é contrato REAL, não palpite — o cliente oficial o usa.

## O que mais vale lembrar

- Sondar GET desconhecido é seguro (read-only); POST só depois de extraído do cliente.
- Campos como `price`/`currency` no corpo costumam ser trava do servidor: preço mudou,
  a ação é recusada — replique a mesma trava na sua rota.
- Vale para qualquer SPA: o minificado preserva strings de URL e shape de objeto.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[Quando a REST não expõe o dado, o WebSocket do mesmo sistema entrega]]
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
