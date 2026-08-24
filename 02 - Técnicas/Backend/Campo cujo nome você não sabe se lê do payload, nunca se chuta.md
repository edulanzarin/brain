---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-24
---

# Campo cujo nome você não sabe se lê do payload, nunca se chuta

> Você provou que o recurso existe mas não achou o nome do campo. O chute parece
> barato porque a API "vai reclamar se estiver errado". Não vai: a maioria ignora
> chave desconhecida e devolve 200. O chute errado não falha — ele *passa*.

## O problema

Uma API tolerante transforma palpite errado em **sucesso falso**. O PATCH sai
com `autoPotionId`, o servidor descarta a chave que não conhece, responde 200
com o estado inalterado, e a tela mostra a escolha aplicada. Nada quebra. O
usuário configura, confia, e o comportamento nunca muda — sem erro, sem log,
sem nada para investigar.

É pior que não ter o recurso: um seletor que não faz nada é uma mentira que o
próprio produto conta, e ela sobrevive a qualquer teste que só verifique o
status da resposta.

## A solução

O nome do campo é **dado da fonte**, então leia da fonte. A conta já devolve o
objeto inteiro no GET: procure nele a chave pela FORMA, e escreva de volta na
mesma chave que leu.

```ts
const CHAVE_POCAO = /potion.*id$/i;

function acharCampoPocao(c: Record<string, unknown>): string | null {
  for (const k of Object.keys(c)) {
    if (!CHAVE_POCAO.test(k)) continue;
    const v = c[k];
    if (typeof v === "number" || v === null) return k;   // a forma confirma
  }
  return null;                                            // não achou: diga isso
}
```

Três propriedades que fazem isso valer:

- **Auto-corrige.** O servidor renomeou o campo? A leitura seguinte acha o nome
  novo. Uma constante fixa quebraria em silêncio, do mesmo jeito do chute.
- **O `null` é resposta.** Não achar não vira zero nem vira palpite: o controle
  não aparece, e a tela diz que esta conta não trouxe o campo. Ver
  [[Ausência de leitura cai no valor que dispara a ação]].
- **Continua lista branca.** O campo entra pela forma (casa o padrão, é numérico),
  não pela confiança — a validação de escrita não afrouxa, só passa a aceitar uma
  regra além da tabela fixa.

## O que mais vale lembrar

- **Confirme pelo estado, não pelo status.** A resposta do PATCH traz o objeto
  novo: compare o que voltou com o que foi pedido. É a mesma disciplina de
  [[Confirme a mutação pelo estado que ela deixa, não pelo ack que pode não chegar]].
- **A forma é parte do reconhecimento.** Casar só pelo nome aceitaria
  `autoPotionThreshold` como se fosse um id. Nome mais tipo erram junto muito
  menos que nome sozinho.
- **Não confunda com aceitar qualquer coisa.** Isto vale para UM campo cuja
  existência já foi comprovada por outra evidência (a i18n do cliente, a tela do
  produto). Sem essa prova, não há o que descobrir — há o que investigar.

## Conexões
- Princípio: [[Peça o que a fonte mostra, não o que você precisa]]
- Irmã: [[O bundle público do cliente entrega o contrato da API sem documentação]] ·
  [[Ausência de leitura cai no valor que dispara a ação]] ·
  [[Confirme a mutação pelo estado que ela deixa, não pelo ack que pode não chegar]] ·
  [[Comando que responde ok e não muda nada tem pré-condição de estado]]
- Visto em: [[piwdex2]]
- Mapa: [[Backend]]
