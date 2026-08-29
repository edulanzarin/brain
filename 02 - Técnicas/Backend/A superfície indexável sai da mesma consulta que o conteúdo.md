---
tags: [tipo/atomica, camada/padrao, dev/backend]
criado: 2026-08-29
---

# A superfície indexável sai da mesma consulta que o conteúdo

> Num site de diretório — cidades, categorias, marcas —, a tentação é gerar uma
> página por linha da tabela de lugares. Não: gere por **lugar que tem conteúdo
> dentro**, e faça a rota, o índice e o sitemap saírem da mesma consulta.

## O problema

O cadastro tem 60 cidades porque o formulário precisa da lista inteira. Se cada
cidade virar página, o site nasce com 58 páginas idênticas dizendo "nenhum
resultado" e 2 com conteúdo. Para o buscador isso é um site de conteúdo raso, e
a punição não fica só nas páginas vazias: ela pesa no domínio.

O sitemap agrava. Ele é uma **promessa**: cada URL listada tem algo lá. Listar
o vazio queima o orçamento de rastreio e ensina o robô a não confiar na lista.

## A forma

Uma função só responde "onde existe conteúdo?", e as três superfícies a
consomem:

```ts
export async function cidadesComAnuncio() {
  const grupos = await prisma.perfil.groupBy({
    by: ["cidadeId"],
    where: { status: "ATIVO" },
    _count: { _all: true },
  });
  ...
}
```

- **A rota** `/[uf]/[cidade]`: sem anúncio, `notFound()`. Não é página de estado
  vazio — é 404, porque a página não existe mesmo.
- **O índice** `/cidades`: lista o que a consulta devolveu.
- **O sitemap**: itera a mesma consulta.

Ter cidade no banco deixa de significar ter página. A cidade entra sozinha no
dia em que alguém publica nela, e sai sozinha quando o último anúncio sai.

## O que mais vale lembrar

- **Filtro não muda a existência da página.** Se a cidade tem 12 anúncios e o
  filtro não casa com nenhum, ali é estado vazio com o filtro na tela, não 404.
  A pergunta do 404 é sobre o lugar, não sobre a seleção.
- **Ordenação e paginação não geram URL canônica nova**: é o mesmo conteúdo
  remexido. O canonical aponta para o endereço sem eles.
- **Registro fora do ar sai do índice mas mantém o link seguido**
  (`noindex, follow`): o robô continua encontrando a listagem por ele.
- Isto vale além de SEO. É a mesma ideia de não oferecer um filtro que não
  devolve nada — [[Tela que abre vazia tem que ensinar, tela que abre cheia não]].

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[Página que consulta o banco não pode nascer no build]]
- Visto em: [[Privello]]
- Mapa: [[Backend]]
