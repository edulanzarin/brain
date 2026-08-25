---
tags: [tipo/atomica, camada/padrao, design, armadilha]
criado: 2026-08-24
---

# Faixa que sangra estoura pela barra de rolagem, e o corte é na raiz

> Seção que vai de borda a borda da janela nasce mais larga do que o lugar onde
> cabe, e a sobra vira um fiapo de barra horizontal na página inteira. A conta erra
> por um valor exato: a largura da barra de rolagem.

## Por que ela estoura

A saída padrão pra sangrar pra fora do container é `left: 50%; width: 100vw;
transform: translateX(-50%)`. O problema é o `100vw`: ele mede a **janela inteira**,
com a canaleta da barra junto. A largura útil da página é a janela **menos** a barra.

Quem reserva a canaleta sempre — `scrollbar-gutter: stable`, que existe pra a grade
não pular 15px quando a barra aparece e some entre páginas — paga isso em toda
faixa: ela sai larga pela diferença, metade pra cada lado. A metade de dentro o
navegador ignora; a de fora vira rolagem horizontal.

O sintoma engana porque é **pequeno e global**: não é uma seção quebrada, é a página
toda deslizando alguns pixels. Medir resolve em um minuto — varra o DOM e compare
`getBoundingClientRect().right` de tudo com a largura do `body` (o `body` tem a
largura útil; `documentElement.clientWidth` pode devolver a janela com canaleta e
mentir na comparação). Se todo estouro der o mesmo número, é este.

## O corte vai na raiz, e não no ancestral mais próximo

O instinto é `overflow-x` no pai da faixa. **Isso mata o efeito**: o pai é
exatamente o container de largura máxima ([[Container tem largura máxima e respiro
constante]]) do qual a faixa precisa escapar — cortar ali devolve a seção pro
container. O único ancestral largo o bastante é o elemento raiz.

E o valor é `clip`, não `hidden`:

- `hidden` transforma o elemento em **container de rolagem**, e aí todo
  `position: sticky` de dentro para de grudar — navegação, cabeçalho de tabela,
  trilho de filtro ([[Sticky gruda no container que rola, não na janela]]);
- `clip` corta sem criar rolagem nenhuma. O eixo Y continua livre.

```css
html {
  scrollbar-gutter: stable;
  overflow-x: clip;
}
```

## O preço do corte: estouro para de avisar

Antes do `clip`, qualquer elemento largo demais se anunciava sozinho — a barra
horizontal aparecia e alguém reclamava. Depois dele, **o mesmo estouro passa a
ser silencioso**: o conteúdo é aparado na borda da janela e some. Numa tela larga
ninguém nota; num telefone, um campo do formulário fica pela metade e a pessoa
não tem como saber que existe mais coisa ali.

Foi exatamente o que aconteceu no mesmo produto, na mesma semana: um cabeçalho de
slot que não podia quebrar pedia 340px de largura mínima, e num aparelho de 390 a
caixa inteira saía cortada — sem barra, sem aviso, sem erro.

Então o corte na raiz **exige medição no lugar do sintoma**. Vale rodar a mesma
varredura de larguras (360, 390, 768, 1024, 1440) ao terminar qualquer tela nova:

```js
const bw = document.body.getBoundingClientRect().width;
[...document.querySelectorAll("*")].filter(el => {
  const r = el.getBoundingClientRect();
  return r.width > 0 && r.right > bw + 0.5;
});
```

## Quem estoura quase sempre é item de grid ou de flex

`min-width: auto` é o valor inicial de item de grid e de flex, e ele significa
"não encolha abaixo do meu **min-content**". O item então **transborda a trilha**
em vez de caber nela — o container fica com 330px e o filho, com 366.

Duas causas cobrem a maioria dos casos:

- **linha que não pode quebrar** — um cabeçalho `flex` com rótulo e dois botões
  soma tudo no min-content. `flex-wrap` resolve e custa uma linha de altura;
- **grade de campos com colunas demais cedo demais** — três campos de número
  pedem mais largura do que um telefone tem. Abra em duas colunas e suba a três
  num breakpoint.

`min-w-0` no item é a outra saída, e serve quando o conteúdo pode truncar
(texto). Não serve quando o conteúdo é um controle que precisa da largura.

## O que não resolve

Nenhuma unidade de CSS devolve "janela menos a barra": `dvw` inclui a canaleta do
mesmo jeito. Descontar um valor fixo (`calc(100vw - 10px)`) só funciona enquanto a
barra tiver a largura que você estilizou — em Firefox, ou com barra do sistema, o
número é outro e o erro troca de sinal. Cortar na raiz não depende de saber a
medida.

## Conexões
- Princípio: [[Container tem largura máxima e respiro constante]]
- Depende de: [[Sticky gruda no container que rola, não na janela]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]]
