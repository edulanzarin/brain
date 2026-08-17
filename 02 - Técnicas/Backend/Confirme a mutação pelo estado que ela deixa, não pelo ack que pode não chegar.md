---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-17
---

# Confirme a mutação pelo estado que ela deixa, não pelo ack que pode não chegar

> Ao mandar um comando que muta o outro lado (WebSocket, fila, RPC fire-and-forget),
> a tentação é esperar o **ack/echo** daquele comando pra dizer "deu certo". Mas o ack é
> um sinal do transporte: pode não vir, vir com outro nome, ou o servidor pode aplicar a
> mutação sem ecoar. Amarrar o sucesso ao ack dá **falso timeout** — a mutação aconteceu,
> mas você não recebeu o carimbo e reportou erro.

A confirmação robusta observa o **efeito**, não o sinal: depois de mandar o comando, peça
o estado (um `get`/refresh) e verifique que o estado agora reflete a mutação. É a mesma
verdade que o resto do sistema lê, então é a prova real. O ack, se vier, é atalho bem-vindo
— mas não a única porta de saída.

Regra prática: `sucesso = (echo do comando) OU (releitura do estado mostra o efeito)`;
`timeout` só se **nenhum** dos dois aconteceu na janela.

## Visto em

No piwdex, o "Usar este" trocava o pokémon líder por `poke-summon` num socket one-shot e
só resolvia com o echo `poke-summon`. O echo nem sempre chegava -> `502` após o timeout,
mesmo com a troca feita. A correção: após o summon, mandar `pokes-get` e confirmar quando o
alvo volta `leader:true` na lista — o efeito observado. O echo virou caminho rápido
opcional, não o único.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[Persistir a mensagem não espera a entrega, a entrega é status]]
- Irmã: [[Config que a sessão cacheia no init não vê a escrita no backend, reaplique na mesma conexão]]
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
