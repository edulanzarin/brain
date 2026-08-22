---
tags: [tipo/atomica, camada/padrao, design, dev/frontend]
criado: 2026-08-22
---

# Faixa de topo de ferramenta é chegada, não rótulo

> Um `<h1>` com o nome responde "como isso se chama". Quem acabou de abrir a tela
> tem outras três perguntas antes dessa, e é a faixa de topo que as responde.

## O sintoma

Seis ferramentas, seis telas que começavam com arte de 44px mais a palavra numa
linha solta. Abrir a Hunt e abrir a Meta produziam a **mesma imagem** com uma palavra
e uma cor de texto trocadas. A tela que faz a conta mais pesada do site abria
parecendo cabeçalho de documento.

## As quatro perguntas, na ordem em que se leem

1. **Onde eu estou** — a arte em tamanho de figura (~88px), e a cor da ferramenta
   pintando a faixa inteira, não só o título.
2. **Como isso se chama** — o título aceso, na cor.
3. **O que isso faz** — uma frase. Não o parágrafo da home: uma frase, porque a
   ferramenta já está na frente da pessoa.
4. **Qual o tamanho disso** — a contagem real do catálogo ("342 alvos", "482
   espécies"). Prova a ferramenta antes de qualquer rolagem, e é o único número da
   faixa que muda quando o jogo publica patch.

## Toda a decoração sai de uma variável

`--tint` é escrita uma vez pelo componente e alimenta wash, grade, facho,
cantoneira, halo, título e o fio de baixo. Não há um hex no arquivo, e trocar a cor
no registro repinta a faixa inteira.

```tsx
<header className="hero panel" style={{ "--tint": f.cor }}>
```

```css
.hero { position: relative; overflow: hidden; isolation: isolate; }
.hero::before {           /* o wash entrando pelo canto onde a leitura começa */
  content: ""; position: absolute; inset: 0; z-index: -1;
  background: radial-gradient(720px 280px at 4% 0%,
    color-mix(in oklab, var(--tint) 30%, transparent), transparent 72%);
}
```

`isolation: isolate` não é opcional: sem ela o `z-index: -1` escapa pro contexto do
pai e as camadas somem **atrás da página** em vez de ficarem atrás do conteúdo.

## Identidade de ferramenta é dado, não JSX

Nome, cor, arte, glifo e frase moravam em três lugares — a home, a navegação e a
chamada de cada página. Identidade repetida em três lugares é identidade que
envelhece em dois. Um registro (`lib/ferramentas.ts`) abastece os três, e a ordem da
lista passa a ser a ordem da navegação.

Efeito colateral que valeu sozinho: a navegação ganhou o glifo e a cor que só a home
tinha. Seis palavras curtas em caixa alta, no mesmo tom, viram uma fileira de manchas
indistinguíveis — com a silhueta na frente, a palavra vira confirmação.

## O que só aparece montando

**Duas animações no mesmo elemento significam uma só.** `class="anim-in anim-float"`
não compõe: as duas escrevem a propriedade `animation` e a última do arquivo CSS
vence, silenciosamente. Quem entra e quem flutua têm que ser dois elementos
aninhados — o invólucro entra, o filho flutua.

E a entrada em cascata sai de uma variável por elemento (`--d`), não de quatro
classes com durações diferentes: é a mesma animação, deslocada, e a faixa se **monta**
em vez de aparecer. Junto vem a armadilha de
[[Reduzir movimento tem que zerar o atraso, não só a duração]].

## Conexões
- Princípios: [[Hierarquia por superfície, não por borda]] ·
  [[Token semântico em vez de valor literal]]
- Irmã: [[Manual de ferramenta é resumo visível com passo a passo sob demanda]] ·
  [[Arte de ícone se julga no tamanho de uso, e o acento é a massa]] ·
  [[Sobre arte de fundo, a chrome também tem piso de opacidade]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]]
