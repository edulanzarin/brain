---
tags: [tipo/atomica, camada/padrao, armadilha, dev/frontend]
criado: 2026-08-30
---

# Segmento de URL que começa com @ não chega ao App Router

> `/@duda` devolve 404 antes de qualquer página rodar. No App Router o `@` é a
> marca de rota paralela, e a reserva vale também no endereço — não só no nome
> da pasta.

## O problema

Perfil com @ tipo Instagram convida a pôr o @ no endereço:
`site.com/rs/porto-alegre/@duda`. Parece só pontuação, o `@` é caractere válido
em caminho de URL pela RFC 3986, e TikTok e Threads fazem exatamente isso.

No Next 16 (App Router) não funciona. O roteador não casa o segmento com
`[arroba]` e responde 404 sem chamar a página. Testado nas duas formas:

```
/rs/porto-alegre/@duda      → 404   (a página não roda)
/rs/porto-alegre/%40duda    → 404   (codificado também não)
/rs/porto-alegre/duda       → 200
```

O jeito de ter certeza de que a página não rodou é pôr um `redirect()` no
começo dela: se o endereço sem `@` redireciona e o com `@` devolve 404, quem
recusou foi o roteador, não o seu código.

## A solução

**A arroba é da tela; o endereço leva o @ sem ela** — que é, aliás, o que o
Instagram faz: mostra `@duda`, endereça `/duda`. O componente que escreve o @
põe a arroba na renderização, e o construtor de endereço não põe.

De quebra, o atalho curto passa a caber na raiz do site. `/duda` e `/rs` têm a
mesma forma, e o que separa os dois é o tamanho: sigla de estado tem exatamente
duas letras, @ tem no mínimo três. A validação do @ é o que garante que nenhum
handle consegue parecer um estado.

```tsx
// a mesma rota [uf] atende os dois, sem ambiguidade
function ehAtalhoDePerfil(uf: string) {
    return uf.length !== 2;
}
```

Não adianta tentar abrir uma rota irmã `@[arroba]` ao lado de `[uf]`: pasta que
começa com `@` é slot de rota paralela, e dois segmentos dinâmicos irmãos com
nomes diferentes não coexistem no App Router.

## O que mais vale lembrar

Só apareceu no build de produção rodando de verdade. O servidor de `dev` estava
com o cliente Prisma velho na memória e devolvia 500 antes de chegar no
roteador, o que escondeu o 404 — o diagnóstico veio de subir a build numa porta
separada e bater nas duas formas do endereço lado a lado.

## Conexões
- Folha isolada por enquanto: falta o princípio que cubra "vocabulário reservado
  da ferramenta não é reutilizável no seu domínio".
- Ver também: [[Identificador que já circulou não é mais seu para mudar]] ·
  [[Verificar no build de produção, não só em dev]]
- Visto em: [[Privello]]
- Mapa: [[Frontend]]
