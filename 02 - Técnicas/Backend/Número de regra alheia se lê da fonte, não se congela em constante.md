---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-20
---

# Número de regra alheia se lê da fonte, não se congela em constante

> Quando a regra é de outro sistema, o número que você observou vale para o dia em que
> observou. Congelado numa constante, ele vira fato aos seus olhos e mentira aos olhos da
> fonte — e o erro não aparece como falha, só como recomendação errada com números certos.

## O problema

Um sistema de terceiro anuncia um número na tela: um bônus de 50%, um limite de 100
itens, uma taxa de 2%. Você anota, escreve `const BONUS = 0.5` e segue. A constante
atravessa o código com cara de constante física, e ninguém mais pergunta de onde veio.

Quando a fonte muda o número — ou quando a observação estava errada desde o começo — nada
quebra. O cálculo continua rodando, o teste continua passando, a tela continua exibindo.
Só a **conclusão** muda de lado: no piwdex o bônus do Tipo do Dia estava anotado como
+50% quando o jogo pagava +20%, então a ferramenta prometia 2,5x o ganho real e mandava
priorizar um alvo que não compensava.

O agravante é a **cópia**. O número quase nunca fica num lugar só: a constante alimenta o
cálculo e a tela escreve `+50%` na etiqueta. As duas cópias divergem em silêncio, e a
etiqueta chumbada é justamente a que dá confiança no número errado.

## A solução

Três movimentos, nesta ordem:

1. **Procure o número na API antes de anotá-lo da tela.** O que o sistema mostra ao
   usuário costuma estar num endpoint. Vale sondar: no piwdex, `/api/game/boosts`
   respondia 401 — e 401 é a melhor resposta possível numa sondagem, porque significa
   "existe, e é seu se você tiver token".
2. **Leia da fonte em tempo de execução, com a constante como padrão.** Quem tem conta
   conectada recebe o número do dia; quem não tem cai no último valor conhecido. A
   constante deixa de ser verdade e vira **reserva** — e o comentário dela diz isso.
3. **A tela lê a mesma origem do cálculo.** Nenhum literal na interface. Foi a etiqueta
   chumbada que fez o valor errado sobreviver a uma revisão inteira.

```ts
// reserva de quem não tem a fonte; o valor do dia vem do jogo
export const TYPE_DAY_BONUS = 0.2;
const pct = bonus.typeDayPct ?? TYPE_DAY_BONUS;   // cálculo
<span>+{Math.round(pct * 100)}%</span>            // tela, mesma origem
```

## O que mais vale lembrar

- **Marque a data e o método na constante.** "Lido com token real em ago/2026" diz muito
  mais que o número sozinho: quem revisar sabe se vale reconferir.
- **Cache curto, não cache eterno.** Um número que muda todo dia se cacheia por minutos,
  e nunca por processo — senão o servidor que subiu ontem responde o bônus de ontem.
- Isto não vale só para jogo: preço de gateway, cota de API, limite de anexo, prazo de
  compensação. Toda regra de terceiro é versionada por eles, não por você.

## Conexões
- Princípio: [[A definição em dado dirige o comportamento, não um caso no código]]
- Irmã: [[Quando o campo numérico vem zerado, o número está na frase]]
- Parente: [[O bundle público do cliente entrega o contrato da API sem documentação]] · [[Estimativa desmentida pela realidade vira veto temporário do motor]]
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
