---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-09-05
---

# Metadata do Next não funde o aninhado, e o que herda vaza

> O objeto `metadata` do App Router parece herança normal: o layout põe o
> comum, a página põe o específico. Não é. Ele funde **raso** — e as três
> consequências disso não dão erro, não aparecem no build, e só se descobrem
> olhando o HTML ou colando o link numa conversa.

## As três faces

**1. Declarar o filho apaga o pai inteiro.** O layout define
`openGraph: { siteName, locale, type }`. A página define
`openGraph: { title }`. O resultado não tem `siteName` nem `locale`: o objeto
foi substituído, não mesclado. Espalhado por vinte páginas, viram vinte cartões
de compartilhamento diferentes, cada um faltando um pedaço diferente.

**2. O que você não declara desce sozinho.** `alternates` é herdado. Um
`alternates: { canonical: "/" }` escrito no layout raiz por conveniência faz
**toda** página sem canônico próprio apontar para a home — que é a forma mais
rápida de tirar o site inteiro do índice sem escrever nada errado.

**3. Declarar o campo desliga a convenção de arquivo.** Existindo
`app/opengraph-image.tsx`, o Next encaixa a imagem sozinho. Mas quando a página
declara `openGraph`, ele para de encaixar — e `images: undefined` conta como
declarar. A home fica sem cartão nenhum, e isso não aparece em lugar nenhum
além do WhatsApp de quem colou o link.

## A forma

Uma função constrói o objeto inteiro, e nenhuma página escreve `openGraph` à
mão:

```ts
export function metaDaPagina({ titulo, descricao, caminho, canonico, imagem }) {
  const url = absoluto(canonico ?? caminho);
  return {
    title: titulo,
    alternates: { canonical: url },          // sempre explícito, nunca herdado
    openGraph: { type, siteName, locale, url, title, description, images: [...] },
    twitter: { card: "summary_large_image", ... },
  };
}
```

Duas decisões que saem disso:

- **O canônico nunca fica no layout raiz.** Ele é sempre da página, mesmo
  quando é óbvio.
- **A imagem padrão é apontada à mão** (`/opengraph-image`), em vez de deixada
  por convenção. Custa uma linha e não depende de um comportamento de merge que
  nem sempre é o que se espera.

## Como conferir

Não confie no tipo nem no build. `curl` na página e olhe a saída:

```bash
curl -s http://localhost:4075/ | grep -oE '<link rel="canonical"[^>]*>|<meta property="og:[^"]*"[^>]*>'
```

Foi assim que o `og:image` faltando apareceu — depois de o TypeScript, o lint e
o build passarem os três.

## Conexões
- Princípio: [[Dado escrito por dois caminhos precisa de uma regra só, fora dos dois]]
  — vinte páginas montando o mesmo cartão é o caso; a saída foi a regra única
  fora delas.
- Irmã: [[Filtro é ferramenta, recorte é página]] — quem decide o que vai no
  canônico que esta nota garante que existe.
- Visto em: [[Privello]]
- Mapa: [[Frontend]]
