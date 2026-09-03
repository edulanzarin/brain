---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-09-03
---

# Slug de asset ganha tipo derivado do manifesto gerado

> Se o componente aceita `arte: string`, passar o asset da forma errada só aparece
> na tela — e aparece como página branca, no passo 3 de um formulário.

## O erro que motivou

Dois componentes, duas formas de arquivo: `<Sprite>` espera folha de ator (4
direções × N quadros); `<Icone>` espera imagem única. Passei `"natureza/katon"` —
que é um pergaminho, imagem única — para o `<Sprite>`. O acessador lançou, e a tela
morreu na etapa de escolher chakra. Tipo `string` nos dois lados não tinha como
avisar.

## A correção

O manifesto de arte já é **gerado** e importado como JSON. Com `resolveJsonModule`,
o TypeScript infere a forma de cada entrada — então dá para separar os slugs por
forma, no tipo:

```ts
import manifesto from "./arte.json";
type Bruto = typeof manifesto;

export type SlugAtor = {
  [K in keyof Bruto]: Bruto[K] extends { folha: string } ? K : never;
}[keyof Bruto];

export type SlugFolha = {
  [K in keyof Bruto]: Bruto[K] extends { arquivo: string } ? K : never;
}[keyof Bruto];
```

`<Sprite arte: SlugAtor>`, `<Icone arte: SlugFolha>`, e o campo `arte` do catálogo
tipado por forma. O mesmo erro passou a ser:

```
Type '"natureza/katon" | ... ' is not assignable to type 'SlugAtor'.
```

Nenhum arquivo novo de configuração: o tipo saiu do dado que já existia.

## O tipo cobre literal; o catálogo cobre o resto

A união pega o slug **escrito no componente**. Slug que vem de campo do catálogo o
compilador também pega, desde que o campo esteja tipado. O que sobra — slug montado
em tempo de execução — vira conferência no relatório de verificação, percorrendo o
catálogo e chamando o acessador de cada forma.

## Onde mora o tipo

Ao lado do manifesto, na camada de **dado** — não na camada de acesso. Assim o
catálogo e o núcleo podem usar sem inverter a direção da dependência, porque o
arquivo não importa nada além do JSON.

## Conexões
- Princípio: [[Peça de mentira que não se anuncia vira fundação de coisa real]]
- Irmã: [[Número que depende de outros números se declara em intenção e se deriva por comando]]
- Visto em: [[naruto-idle]]
- Mapa: [[Frontend]]
