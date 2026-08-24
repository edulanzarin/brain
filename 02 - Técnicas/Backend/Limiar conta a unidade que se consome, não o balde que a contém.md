---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-23
---

# Limiar conta a unidade que se consome, não o balde que a contém

> Reposição automática compara um estoque com um piso. O estoque quase sempre é
> escrito como soma de uma categoria — "quantas bolas", "quantas poções", "quanto
> de crédito" — e o consumo quase nunca é da categoria: é de UM item. Enquanto a
> soma estiver alta, o piso nunca dispara, e a coisa que acabou continua acabada.

## O caso

No [[piwdex2]] o robô tinha a reposição de pokébola ligada, piso em 150, e não
comprava nada. A bolsa tinha 555 Poké Ball e 41 Idle Ball — 596 no total, muito
acima do piso. Só que o auto-catch do jogo estava configurado em **Ultra Ball**,
e de Ultra Ball havia **zero**.

Os dois lados estavam certos isoladamente e o sistema estava parado: o piso
media "tem bola?" e o jogo consumia "tem Ultra Ball?". Nenhum erro, nenhum log,
nenhuma exceção — só um robô que não captura e uma reposição que se recusa a
agir, cada um pelo motivo do outro.

A mesma armadilha estava armada nas poções: o piso somava a categoria `heal`
inteira, Potion fraca junto com Hyper Potion.

## A regra

Quem escolhe o item escolhe também a régua:

```ts
function estoqueDoAlvo(itens, escolhido) {
  if (escolhido) {
    const i = itens.find((x) => x.id === escolhido);
    if (!i) return 0;                      // escolhi o que nem tenho: é zero
    return i.infinita ? Infinity : i.quantidade;
  }
  // sem escolha, a pergunta volta a ser "tem alguma?" — e aí a soma está certa
  return itens.reduce((s, i) => (i.infinita ? s : s + i.quantidade), 0);
}
```

O ponto não é "sempre conte por item". É que **o piso e o consumo têm que medir
a mesma coisa**. Sem item escolhido, a soma é a régua honesta; com item
escolhido, ela vira um número grande que responde outra pergunta.

O caso infinito merece o ramo próprio: item que não acaba nunca fura piso, e
somá-lo junto dos finitos disfarça o estoque real dos outros.

## O que mais vale lembrar

- **O sintoma é inércia, não erro.** Automação que decide por limiar falha
  ficando quieta. Não há stack trace de "não comprei" — o único jeito de pegar é
  a tela mostrar o número que a decisão usou.
- **Uma contagem só, servindo tela e motor.** Aqui a função vive no módulo de
  contrato (o que o cliente pode importar) justamente para que o cartão mostre o
  MESMO número que o motor compara. Duas implementações dariam uma tela dizendo
  596 e um robô decidindo por 0 — e a que ninguém revisa é a que executa.
- **Onde mais isso aparece:** ponto de reposição por categoria de SKU em vez de
  por SKU; cota de disco por volume quando o que enche é uma partição; limite de
  conexões no total quando o gargalo é por host; crédito de API somado entre
  planos quando a chamada consome o de um.
- A pergunta de teste é curta: **"o que o sistema consome é o que eu estou
  contando?"**. Se a resposta precisa de um "mais ou menos", já está errada.

## Conexões
- Princípio: [[Espelhar por balde esconde item no lugar errado]]
- Irmã: [[Ausência de leitura cai no valor que dispara a ação]] ·
  [[Duas listas parecidas respondem perguntas diferentes, e a errada some com o item]] ·
  [[Limiar em grandeza contínua vira degrau, e o degrau decide a ordem]]
- Visto em: [[piwdex2]]
- Mapa: [[Backend]]
