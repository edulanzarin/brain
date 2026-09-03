---
tags: [tipo/atomica, camada/padrao, design, dev/frontend, armadilha]
criado: 2026-09-03
---

# Papel não herda o tema nem a moldura da tela

> Imprimir um app de tema escuro sem preparar a impressão dá página em branco: o
> navegador não pinta o fundo escuro, mas pinta o texto claro. E mesmo legível, o
> papel sai sem empresa, sem período e sem os filtros — tudo isso morava na
> chrome, que não vai junto.

## O problema

Salvar em PDF pelo diálogo do navegador é o caminho mais barato de exportação:
`window.print()`, e a tela decide o que sai via `@media print`. O barato tem duas
armadilhas, e nenhuma aparece em desenvolvimento se o tema estiver claro.

**A paleta.** Fundo é decoração para o navegador: por padrão ele não imprime cor
de fundo, só o conteúdo. Num tema escuro, o conteúdo é claro. Resultado: branco
no branco, uma folha que parece vazia.

**A moldura.** O que dá contexto ao relatório na tela — empresa selecionada,
período, filtros aplicados — quase sempre mora no cabeçalho e na barra de
filtros, fora do recorte que vai pro papel. O PDF sai com uma tabela e um rodapé
escrito "Total da empresa" que, filtrado, é mentira para quem ler dali a meses.

## A solução

**Tratar papel como um quarto tema**, no CSS global e não na tela. Se a cor já é
token semântico, é um bloco só, e vale para toda tela que imprime:

```css
@media print {
  :root, :root[data-theme="light"], :root[data-theme="dark"] {
    color-scheme: light;
    --surface: #fff; --ink: #000; --muted: #55555f;
    --hairline: rgba(0, 0, 0, 0.25);
    --shadow-1: none; --shadow-2: none;   /* sombra é tinta desperdiçada */
  }
  @page { margin: 12mm; }

  .no-print { display: none !important; }      /* controle não vai pro papel */
  .print-solto { max-height: none !important; overflow: visible !important; }
  thead { display: table-header-group; }        /* cabeçalho repete por página */
  tr { break-inside: avoid; }                   /* linha não parte ao meio */
}
```

**Dar ao relatório um cabeçalho que só existe no papel** — `hidden print:block` —
com empresa, período e a lista dos filtros ativos. É a moldura viajando junto.

Selecionar o que imprime continua sendo da tela:

```css
body * { visibility: hidden !important; }
#relatorio, #relatorio * { visibility: visible !important; }
#relatorio { position: absolute; left: 0; top: 0; width: 100%; }
```

## O que mais vale lembrar

- `visibility` em vez de `display` preserva o layout de dentro do recorte;
  `position: absolute` com `left/top/width` (não `inset: 0`) evita altura presa ao
  contêiner quando o relatório tem várias páginas.
- **Toda tabela rolável some no papel.** `max-height` + `overflow` cortam a lista
  em quarenta linhas; sem a classe que solta a altura, o PDF exporta o que coube
  na tela e ninguém percebe — parece completo.
- Deixar a barra de filtros FORA do contêiner de impressão já a esconde: não
  precisa de `.no-print` no que nasceu fora do recorte.
- Não force `print-color-adjust: exact` para "manter o visual": é o contrário do
  que se quer, imprime o fundo escuro inteiro.
- Registre a exportação na trilha de auditoria antes de abrir o diálogo — é
  exportação de dado como qualquer outra, mesmo saindo por impressora.

## Conexões
- Princípio: [[Token semântico em vez de valor literal]]
- Irmã: [[Sistema de cores e tema do dashboard]] · [[A tela não afirma mais precisão do que a fonte tem]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Design]]
