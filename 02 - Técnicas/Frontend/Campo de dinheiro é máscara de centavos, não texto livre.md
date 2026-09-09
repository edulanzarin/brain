---
tags: [tipo/atomica, camada/padrao, dev/frontend]
criado: 2026-09-09
---

# Campo de dinheiro é máscara de centavos, não texto livre

> Se o campo nunca aceita um ponto, ninguém precisa decidir se aquele ponto era
> milhar ou decimal.

## O problema

Preço digitado como texto chega em todas as grafias que a pessoa aprendeu na
vida — `1200`, `1.200`, `1.200,00`, `250,50` — e em português o ponto tem dois
papéis. Dá para desempatar pela contagem de casas
([[Ponto em preço brasileiro é ambíguo, e quem desempata é a contagem de casas]]),
mas isso é heurística: o campo continua aceitando qualquer coisa e o acerto
depende de um parser lembrado toda vez que outro formulário nascer.

## A solução

O campo não aceita texto, aceita **dígito**. Cada tecla empurra as casas
decimais e o que aparece é o valor já formatado; o estado é inteiro em centavos
do campo até a coluna do banco.

```ts
// entrada: só os dígitos importam
const digits = raw.replace(/\D/g, "").slice(0, 11);
const cents = digits === "" ? null : Number(digits);

// saída: um formatador só, nunca toFixed (que devolve ponto)
const shown = (cents / 100).toLocaleString("pt-BR", {
    minimumFractionDigits: 2,
    maximumFractionDigits: 2,
});
```

Digitar `95000` mostra `950,00`. Não há string ambígua em lugar nenhum do
caminho porque nunca existiu uma.

## O que mais vale lembrar

- **Colar continua funcionando**: `1.200,00` colado vira os mesmos dígitos e sai
  certo. `1200` colado vira R$ 12,00 — e a tela mostra isso **na hora**, que é a
  diferença entre uma máscara honesta e um parser silencioso.
- **Vazio é nulo, não zero.** "Ainda não informado" e "isento" são estados
  diferentes, e zero é um valor legítimo de honorário.
- **Centavos em inteiro também poupa a borda da API**: o `Decimal` do Prisma sai
  como objeto no JSON e precisa de conversão nas duas pontas; inteiro atravessa.
- A máscara protege a **digitação**. Valor que entra por importação, colagem em
  massa ou API de terceiro não passa por ela — ali continua valendo o parser da
  nota irmã.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]] — "o
  valor é centavo inteiro e sem ambiguidade" vira propriedade do campo, não
  disciplina de quem escreve o próximo formulário.
- Irmã: [[Ponto em preço brasileiro é ambíguo, e quem desempata é a contagem de casas]] ·
  [[Ponto decimal em interface pt-BR afirma outro número]]
- Visto em: [[Cofre Digital]]
- Mapa: [[Frontend]]
