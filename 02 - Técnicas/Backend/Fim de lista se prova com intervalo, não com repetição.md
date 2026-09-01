---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-09-01
---

# Fim de lista se prova com intervalo, não com repetição

> Numa API que responde **vazio sob pressão** em vez de 429, "acabou a lista" e "não te respondo agora" chegam com o mesmo corpo e o mesmo status. Repetir a chamada não desempata: as repetições caem dentro da mesma janela de throttle e voltam vazias também. O que desempata é o **intervalo** entre elas.

## O problema

Três sinais de fim de lista, e os três mentem nesta situação:

1. **Página vazia** — pode ser throttle disfarçado.
2. **Página menor que o tamanho de página** — pode ser fim, ou pode ser como
   aquele endpoint específico pagina.
3. **N vazios seguidos** — parece robusto e não é, se os N acontecem em
   milissegundos.

Medido numa varredura real, a MESMA consulta em três rodadas sem espaçar as
chamadas: **865, 660 e 0 itens**. A rodada que devolveu zero deu três vazios
consecutivos logo na primeira página — a confirmação por contagem passou por
cima e concluiu "lista vazia".

## A solução

**Espaçar.** A mesma consulta, com 1,4 s entre chamadas: **865 itens nas duas
rodadas, idêntico.** O espaçamento é o que torna a leitura determinística; a
contagem de confirmações só ajuda se houver recuo crescente entre as tentativas.

```ts
if (!lote.length) {
  if (++vazias >= LIMITE) break;
  await pausa(3000 * vazias);   // recuo CRESCENTE — sem isto as N tentativas
  pagina--;                     // caem todas na mesma janela de throttle
  continue;
}
vazias = 0;
```

Uma varredura "rápida" dessa API não é rápida: é uma que mente. O custo certo é
o previsível — melhor 46 minutos que fecham do que 10 que truncam em silêncio.

## O critério de parada é POR ENDPOINT, não por API

Na mesma API, medido:

- `/requests/ListAll` — tem **uma página curta no MEIO** da lista, de forma
  reprodutível (duas rodadas idênticas: 865 itens, 45 páginas, a curta sempre no
  mesmo lugar). Parar em "página menor que o tamanho" trunca em ~metade.
- `/deliveries/{id}` — as páginas vêm 50, 50, …, 46. Só a última é curta, e ali
  o critério por tamanho é seguro e economiza duas chamadas por item.

Então não se escolhe UM critério para o cliente inteiro: cada endpoint ganha o
seu, com a medição ao lado no comentário. Generalizar aqui é como o truncamento
entra sem ninguém notar.

## O que mais vale lembrar

- **Confirme o vazio, nunca o cheio.** Página cheia é evidência de si mesma;
  vazia é sempre suspeita.
- **Vazio "confirmado" no meio da varredura merece log**, mesmo quando a
  repetição resolveu: é a medida de quanto a API está engasgando, e é o que
  avisa antes de o ritmo precisar mudar.
- **204 é pior que vazio.** Quando a API usa 204 tanto para "sem novidade"
  quanto para "parâmetro que eu não aceito", nenhum código consegue distinguir —
  a proteção é validar o parâmetro do seu lado antes de mandar.

## Conexões
- Princípio: [[Ausência de leitura cai no valor que dispara a ação]] — página vazia cai em "continuar procurando", nunca em "terminei"
- Irmã: [[Recusa não é falha: contra o não do servidor, insistir é ruído]] — o 429 explícito pede o contrário: recuar. O vazio silencioso é que pede insistir espaçado
- Depende de: [[Chamada externa tem timeout e erro tratado]]
- Visto em: [[Navetech Hub]] — varredura do [[API do Acessórias]]
- Mapa: [[Backend]]
