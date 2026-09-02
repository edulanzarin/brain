---
tags: [tipo/atomica, camada/padrao, dev/frontend, design, armadilha]
criado: 2026-09-02
---

# Utilitária arbitrária com sintaxe densa não atravessa o parser, e some calada

> `bg-[url('data:image/svg+xml,<svg …>')]` não vira CSS. O Tailwind não erra, não
> avisa e não gera a regra: a classe simplesmente não existe na folha. E como a
> peça continua respondendo ao clique e mudando de cor, o que falta passa por
> escolha de design.

## O sintoma

Caixa de marcação escrita em utilitárias, com o visto embutido como data-URI:

```tsx
"checked:bg-acento",
"checked:bg-[url('data:image/svg+xml;utf8,<svg …><polyline points=\"20 6 9 17 4 12\"/></svg>')]"
```

A caixa marcava, ficava azul e **não tinha visto nenhum**. Nada no console, nada
no build. Só se descobriu procurando `polyline` na folha compilada — zero
ocorrências.

O valor arbitrário tem aspas, espaços, `<`, `>` e `#`. O extrator do Tailwind lê
a classe do código-fonte como texto e desiste de casos assim; desistir, para
ele, é não gerar a regra.

## A solução

Valor com sintaxe própria vira **classe de componente**, escrita em CSS de
verdade — onde ele passa pelo parser do CSS, que é quem sabe lê-lo:

```css
@layer components {
  .nx-caixa:checked {
    background-image: url("data:image/svg+xml,%3Csvg…%3E");
    background-size: 12px;
  }
}
```

E a URL vai **percent-encoded**: `%23` para o `#` da cor (cru, ele encerraria a
URL), `%20` para o espaço, `%3C`/`%3E` para os sinais.

## O que mais vale lembrar

- **A verificação não é olhar a tela, é procurar na folha compilada.** Um
  `grep` por um termo do valor (`polyline`, o nome do token) no CSS do build
  responde em segundos, e é o único jeito de ver o que não foi gerado.
- Vale a mesma desconfiança para qualquer arbitrário longo: `content-[…]` com
  aspas, `grid-cols-[…]` com funções aninhadas, seletor arbitrário com vírgula.
- É a mesma família de falha das notas irmãs: no build, **o que some não
  reclama**. Cor que cai para o herdado, prefixo que engole a versão padrão,
  classe que nunca nasce — todas produzem uma tela plausível.

## Conexões
- Princípio: [[Verificar no build de produção, não só em dev]]
- Irmã: [[Token de cor que não existe vira cor herdada, sem erro]] · [[Propriedade com prefixo escrita à mão pode perder a versão padrão no build]] · [[Classes de componente vão em @layer components no Tailwind]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Frontend]]
