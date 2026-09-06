---
tags: [tipo/atomica, camada/padrao, dev/backend]
criado: 2026-09-05
---

# Filtro é ferramenta, recorte é página

> Numa listagem filtrável, parte do que está na URL descreve **um pedaço do
> mercado** e merece endereço no índice; o resto é **ferramenta de quem já
> chegou** e não merece. Tratar os dois igual dá em uma de duas ruínas: ou o
> índice enche de milhares de variações da mesma lista, ou o site desiste de
> páginas que respondem perguntas que as pessoas realmente digitam.

## Como se separa

A pergunta é se **alguém digita aquilo na busca como uma coisa só**.

- "acompanhantes **trans** em Blumenau" — alguém digita. É recorte: página
  própria, com título próprio, H1 próprio, canônico apontando para si e entrada
  no sitemap.
- "acompanhantes em Blumenau **online agora, com vídeo, até R$ 300, ordenado
  por maior preço**" — ninguém digita. É ferramenta: `noindex, follow`, com
  canônico apontando para a limpa.

O `follow` é a metade que se esquece: a página filtrada não deve competir com a
limpa, mas os itens listados nela devem continuar sendo achados por ali.

## Por que a conta é grande

A rota é dinâmica, então toda combinação responde 200. Sem a separação, um site
de duzentas cidades entra no índice como dezenas de milhares de páginas quase
iguais — e o orçamento de rastreamento que o robô gasta remoendo isso é o mesmo
que ele não gasta nas páginas novas. A punição não fica nas variações: pesa no
domínio.

## O que o recorte exige além do canônico

Um recorte indexável é uma página de verdade, e página de verdade tem três
coisas que a variação não tem:

- **Título e H1 próprios.** Com H1 fixo, a página do recorte fica com título de
  busca de uma coisa e cabeçalho de outra.
- **Um caminho de link até ela.** No primeiro corte as abas de recorte eram
  botões com `onClick` empurrando rota — e botão não é caminho. O robô nunca via
  os outros dois recortes, por mais correto que estivesse o canônico. Viraram
  link de verdade, o que de quebra devolveu o botão do meio do mouse.
- **Gente dentro.** Recorte vazio é soft 404 com título de página cheia; ele sai
  do índice e não entra no sitemap.

## O que mais vale lembrar

Isto refina o "ordenação e paginação não geram URL canônica nova" de
[[A superfície indexável sai da mesma consulta que o conteúdo]]: a regra ali é
verdadeira, mas incompleta. Nem todo parâmetro é remexida do mesmo conteúdo —
alguns são um conteúdo diferente.

O teste não é técnico, é de vocabulário: **existe uma frase que descreve esse
subconjunto e que alguém falaria em voz alta?**

## Visto em

No Privello, `/[uf]/[cidade]` e `/[uf]/[cidade]?genero=TRANS` são páginas; tudo
o mais que a barra de filtro liga sai do índice.

## Conexões
- Princípio: [[Estado compartilhável mora na URL]] — o filtro vai para a URL
  porque descreve o que está sendo visto; esta nota é a segunda pergunta, sobre
  o que daquilo merece ser uma página.
- Irmã: [[A superfície indexável sai da mesma consulta que o conteúdo]]
- Visto em: [[Privello]]
- Mapa: [[Backend]]
