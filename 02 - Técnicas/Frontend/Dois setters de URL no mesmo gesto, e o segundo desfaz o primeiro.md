---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-09-02
---

# Dois setters de URL no mesmo gesto, e o segundo desfaz o primeiro

> Escolher a empresa gravava a empresa e limpava a conta — dois setters no mesmo
> clique. Os dois montaram a URL nova a partir da URL VELHA, e o segundo
> publicou uma sem a empresa. A tela voltava a "Selecione a empresa" no mesmo
> gesto em que a empresa foi escolhida.

## O problema

Guardar filtro na URL é a decisão certa (ver
[[Estado compartilhável mora na URL]]), e o jeito natural de embrulhar isso é um
hook por parâmetro:

```ts
const [aba, setAba] = useParametroUrl("aba");
const [empresa, setEmpresa] = useParametroUrl("empresa");
const [conta, setConta] = useParametroUrl("conta");
```

Cada `setX` faz `new URLSearchParams(searchParams)` a partir do que ele leu no
render, aplica a sua chave e navega. Enquanto só um roda por gesto, funciona.

Mas gestos reais mexem em mais de um parâmetro — trocar a empresa **precisa**
limpar a conta, que pertence ao plano dela. Chamados em sequência, os dois leem
a mesmíssima URL antiga (a navegação é assíncrona; nada mudou entre as duas
linhas) e a última navegação vence. O parâmetro que o primeiro gravou nunca
existiu.

O defeito é traiçoeiro porque **a interface parece ignorar o clique**: nenhum
erro, nenhum log, e o campo simplesmente não guarda a escolha.

## A solução

Um setter para vários parâmetros, com UMA navegação:

```ts
const [url, definir] = useParametrosUrl({ aba: "importar", empresa: "", conta: "" });

definir({ empresa: String(codigo), conta: "" }); // uma leitura, uma escrita
```

O hook de um parâmetro continua existindo — escrito **em cima** do de vários, e
não ao lado: duas implementações se separam no primeiro conserto.

## O que mais vale lembrar

- **A regra vale para qualquer estado publicado fora do React**: URL,
  `localStorage`, cookie. O padrão "ler, alterar minha chave, escrever tudo de
  volta" é seguro para uma chave e destrutivo para duas.
- **O sintoma é sempre "o clique não funcionou"**, e por isso a investigação
  começa no componente errado — no combo, no evento, no portal. A causa está no
  handler, uma linha abaixo.

## Conexões
- Princípio: [[Estado compartilhável mora na URL]]
- Irmã: [[Ajustar estado no render é legítimo, empurrar rota não é]] ·
  [[A preferência gravada não é o estado em vigor]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Frontend]]
