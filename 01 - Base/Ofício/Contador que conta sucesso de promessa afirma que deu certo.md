---
tags: [tipo/atomica, camada/principio, armadilha]
criado: 2026-08-24
---

# Contador que conta sucesso de promessa afirma que deu certo

> Um resumo de execução costuma nascer da forma mais barata: rodou N tarefas em
> paralelo, conte quantas não falharam, imprima "N/N ok". O problema é que
> "não falhou" e "fez o trabalho" são coisas diferentes, e a diferença mora
> exatamente onde o resumo deveria ajudar: **desistir cedo também é terminar bem.**

## Como isso engana

Uma função que valida pré-condições e volta antes de agir resolve a promessa com
sucesso. Um `Promise.allSettled` a classifica como `fulfilled`. O contador soma.
O log afirma que tudo correu bem — e é justamente esse log que alguém vai ler
quando o sistema não fizer o que devia.

No [[piwdex2]] o boot religava as sessões do robô e imprimia
`1/1 sessao(oes) retomada(s)`. O único alvo tinha saído na primeira linha, pela
porta do token vencido. Nada foi religado, e a mensagem dizia o contrário. A
investigação foi atrás de um problema de processo (o boot roda em outro
contexto? o `globalThis` é compartilhado?) porque a única evidência disponível
afirmava que a retomada tinha acontecido.

**Um contador errado é pior que nenhum**: sem log, procura-se a causa; com o log
errado, procura-se no lugar errado — e com confiança.

## O que fazer

- **Devolva o desfecho, não `void`.** `"retomada" | "token_vencido" |
  "sem_shard"` custa uma união de strings e transforma o resumo em diagnóstico.
- **Conte por desfecho**, não por ausência de exceção:
  `3 desejadas -> 1 retomada, 2 token_vencido`. Quem lê já sabe o que fazer.
- **Separe "não deu erro" de "fez"** também em relatórios de import, migração,
  sincronização e envio. Todo lugar onde um item pode ser legitimamente pulado é
  um lugar onde este contador mente.
- Antes de investigar um comportamento estranho, **duvide da métrica que o
  descreve**. Se ela foi escrita junto com o código, ela carrega as mesmas
  suposições que o bug.

## Conexões
- Irmã: [[Laço que trata toda falha igual apaga a causa da primeira]] (o mesmo
  apagamento, do outro lado: lá a causa some no retry, aqui some no resumo) ·
  [[Auditar o registro, não só o agregado]]
- Visto em: [[piwdex2]]
- Mapa: [[Base]] · [[Backend]]
