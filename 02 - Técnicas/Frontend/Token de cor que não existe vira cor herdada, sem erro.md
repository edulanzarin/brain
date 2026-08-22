---
tags: [tipo/atomica, camada/padrao, dev/frontend, design, armadilha]
criado: 2026-08-21
---

# Token de cor que não existe vira cor herdada, sem erro

> `color: var(--green)` num projeto que não define `--green` não falha, não avisa e
> não fica sem cor: a declaração vira inválida no tempo de computação e a
> propriedade cai para o valor **herdado**. O texto aparece — na cor errada — e
> nada no console diz que faltou um token.

## O sintoma

No [[piwdex2]] os motores de combate e de meta foram portados do projeto anterior,
e vieram com as tabelas de cor de lá:

```ts
export const RISK_COLOR = { safe: "var(--green)", risky: "var(--yellow)", deadly: "var(--red)" };
export const TIER_COLOR = { S: "var(--yellow)", A: "var(--green)", B: "var(--cyan)", ... };
```

O piwdex2 nomeia os tokens de outro jeito (`--color-ok`, `--color-warn`,
`--color-danger`). Resultado: "SEGURO", "ARRISCADO" e "LETAL" saíam **todos
brancos**, e a escada inteira de tier saía branca. A tela continuava dizendo as
palavras certas e não dava erro nenhum — só tinha parado de dizer o que a cor
dizia. Passou por três revisões de tela até alguém estranhar o branco.

## Por que passa despercebido

- **`var()` desconhecida é erro em tempo de COMPUTAÇÃO**, não de parse: o CSS é
  válido, o devtools mostra a declaração, e o valor computado é o herdado.
- **Herdado costuma ser legível.** Se caísse para transparente, o texto sumia e
  alguém via na hora. Caindo para a cor do texto, fica bonito e errado.
- Cor de dado costuma morar em **TypeScript**, não em CSS — então nenhum lint de
  estilo olha para aquela string.

## O que fazer

- **Ao portar módulo entre projetos, os tokens são fronteira.** Um `grep` por
  `var(--` no que veio junto custa dez segundos e é o único jeito de ver.
- **Fallback quando o token é externo ao módulo**: `var(--color-ok, #46e08a)`. Não
  resolve o nome errado, mas troca "cor errada silenciosa" por "cor certa".
- Se as cores de dado moram em `lib/`, vale um teste que confere se todo token
  citado existe na folha — é uma varredura de duas expressões regulares.

## Conexões
- Princípio: [[Token semântico em vez de valor literal]] · [[Todo estado da tela tem visual]]
- Irmã: [[Acento da interface é um token separado da cor de dado]]
- Visto em: [[piwdex2]]
- Mapa: [[Frontend]]
