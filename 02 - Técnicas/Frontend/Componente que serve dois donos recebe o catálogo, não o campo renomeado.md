---
tags: [tipo/atomica, camada/padrao, dev/frontend]
criado: 2026-08-28
---

# Componente que serve dois donos recebe o catálogo, não o campo renomeado

> Quando um segundo módulo quer o gráfico que o primeiro já tem, há duas saídas: fazer o
> segundo falar a língua do primeiro (renomear o campo dele para `lancamentos`) ou fazer
> o componente parar de ter língua própria. A segunda é a certa — e custa menos do que
> parece.

## O erro tentador

O gráfico do Contábil lê `p.lancamentos`. O Fiscal tem `p.notas`. Renomear `notas` para
`lancamentos` "resolve" em um segundo e é o começo do apodrecimento: o vocabulário do
domínio novo passa a ser refém de quem chegou primeiro, e daqui a dois módulos ninguém
lembra por que a tabela de notas fiscais tem uma coluna chamada lançamentos.

O oposto — copiar o componente e trocar o campo — é pior ainda: a partir daí toda
correção precisa ser feita duas vezes, e uma delas será esquecida.

## O que entra por prop

Três coisas, e não mais do que isso:

- **O catálogo** da dimensão categórica: a lista `{ id, rótulo, cor, descrição }`. É o
  que muda de verdade entre módulos — natureza do lançamento num, espécie da nota no
  outro — e é o que o gráfico usa para empilhar, legendar e colorir. O dono original
  vira o valor **padrão** da prop, então nenhuma chamada existente muda.
- **O acessor** do número: `itens={(e) => e.notas}` em vez de um nome de campo fixo.
  Cada domínio mantém o seu vocabulário e o componente não conhece nenhum.
- **O rótulo do item**: "Lançamentos", "Notas", "Tributo". Aparece no tooltip, no
  título e no eixo; sem ele o componente afirma o vocabulário de um dos dois.

O resto — a forma do gráfico, o vazio, o skeleton, o formato dos números — continua
dentro, que é o motivo de existir um componente.

## A armadilha de tipo (TypeScript)

Para o componente aceitar "um ponto com uma chave por classe do catálogo", o tipo tem de
ganhar índice: `{ bucket: string; total: number; [classe: string]: string | number }`.
E aí o tipo do dono original **para de ser atribuível** — porque o TypeScript só infere
índice implícito em *type alias*, nunca em `interface`.

```ts
// não passa: interface não ganha índice implícito
export interface CtbSeriePonto extends PorClasse { bucket: string; total: number }

// passa: alias ganha
export type CtbSeriePonto = PorClasse & { bucket: string; total: number }
```

Uma palavra. Vale saber antes de concluir que a generalização "não dá" e partir para o
copiar-e-colar.

## Quando NÃO generalizar

Se as colunas da tabela são outras, o vazio é outro e a pergunta é outra, o componente
compartilhado vira um bolo de props opcionais que ninguém entende. Aí são dois
componentes mesmo — o que se compartilha é a camada abaixo (a escada, a paleta, os
formatadores), não a peça montada.

O teste: se ao adicionar o segundo dono aparece mais de um `if (modulo === ...)` dentro
do componente, a peça errada foi compartilhada.

## Conexões
- Princípio: [[Conhecimento pertence à base, não ao projeto]]
- Irmã: [[O que dois módulos compartilham é a query, não a rota]] — o mesmo raciocínio
  do outro lado do fio: lá o compartilhado é a consulta, aqui é a peça de tela.
- Irmã: [[Escada ordinal empresta a forma entre domínios, nunca os cortes]] ·
  [[Chip que serve a duas grandezas declara qual delas mostra]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Frontend]]
