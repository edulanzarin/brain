---
tags: [tipo/atomica, camada/padrao, dev/frontend]
criado: 2026-09-25
---

# Excel e PDF saem da mesma tabela, e o tipo se reconhece no exportador

> Quando pedem "exportar em Excel, PDF ou CSV", a tela não deveria aprender três formatos. Ela entrega cabeçalhos e linhas, como já fazia para o CSV, e um exportador só decide o resto: reconhece número e data na célula, monta a planilha e pagina o PDF.

## O problema

O sistema já exportava CSV em 41 telas, cada uma montando `{ cabecalhos, linhas }` com os valores escritos para o Excel pt-BR: `decimalBR` ("1234,56") e `dataBR` ("19/03/2025"). O pedido de Excel e PDF parecia exigir que cada tela declarasse o tipo de cada coluna, para a planilha gravar número como número. Isso seria mexer em 41 telas e manter o tipo em dois lugares.

## A solução

**O tipo se lê dos formatos que o próprio sistema escreve.** Há só dois, e os dois têm forma inconfundível:

```ts
const DECIMAL_BR = /^-?(\d{1,3}(\.\d{3})+|\d+),\d+$/;   // "1.234,56" ou "1234,56"
const DATA_BR = /^(\d{2})\/(\d{2})\/(\d{4})(?: (\d{2}):(\d{2}))?$/;
```

Número do JavaScript vira número; string que casa vira decimal ou data; o resto é texto. O que só parece número continua texto de propósito: código com zero à esquerda (`0012`), CNPJ, competência (`09/2026`), conta do plano (`1.1.01.001`). A data se confere na volta (`31/02` vira `03/03` no `Date`), para uma string impossível cair como texto e não como data errada.

**A planilha é um zip de seis XML**, e escrever à mão sai menor que qualquer biblioteca de planilha:

- `[Content_Types].xml`, `_rels/.rels`, `xl/workbook.xml`, `xl/_rels/workbook.xml.rels`, `xl/styles.xml` e `xl/worksheets/sheet1.xml`.
- Texto como `inlineStr` (dispensa a tabela de strings compartilhadas).
- Data como serial do Excel, `ms / 86_400_000 + 25_569`, com um `numFmt` `dd/mm/yyyy`.
- Valor com o formato embutido 4 (`#,##0.00`).
- Cabeçalho congelado (`<pane ySplit="1" state="frozen"/>`) e filtro ligado.

O zip vem do `fflate` (`zipSync`).

**O PDF é a mesma tabela paginada** (jsPDF com `jspdf-autotable`):
- título, de onde veio e quantas linhas;
- cabeçalho repetido em cada página;
- coluna numérica alinhada à direita, detectada pelo mesmo reconhecimento;
- rodapé com quem gerou, quando e a página.

As duas bibliotecas entram por `import()` dentro do clique, então a tela com o botão não paga por elas ao abrir.

## O que mais vale lembrar

- **O filtro da planilha precisa do nome oculto `_xlnm._FilterDatabase`** no `workbook.xml` apontando para o mesmo intervalo. Sem ele o Excel abre e oferece "reparar".
- **Um caractere de controle no texto invalida o arquivo inteiro.** O XML 1.0 não aceita `\u0000`–`\u0008`, `\u000B`, `\u000C`, `\u000E`–`\u001F`, nem surrogate solto. Um só, num nome importado, e o Excel recusa. Limpa-se no escape.
- **Nome de aba:** até 31 caracteres, sem `[]:*?/\`, nunca vazio.
- **`Math.max(...linhas)` estoura** com dezenas de milhares de linhas (limite de argumentos). `reduce` resolve.
- **As fontes-padrão do PDF (Helvetica) escrevem Latin-1.** Acento de português passa; travessão, aspas curvas, reticências e seta viram lixo. Troque pelo equivalente antes de escrever.
- **Sem `compress: true` o jsPDF gera PDF inchado.** No caso daqui, 120 linhas passavam de 300 KB e caíram para 30 KB.
- **"Página X de Y" se escreve depois da tabela**, percorrendo as páginas com `setPage`. O marcador de total do jsPDF (`putTotalPages`) alinha pela largura do marcador, e o número à direita sai torto.
- **Inteiro sai cru no PDF.** Código de empresa, contrato e ano também são inteiros, e "1.402" como código de empresa lê errado. Só o decimal ganha milhar.
- **O PDF tem teto de linhas** (aqui 5.000, uma centena de páginas). Acima disso ele trava a aba montando, e quem precisa de tudo leva a planilha.
- **Sem Excel na máquina, a conferência é em camadas.** Abrir o zip e passar cada parte num parser de XML (`[xml]` do PowerShell) pega o arquivo malformado. O PDF se confere renderizado.
- Conferir que número chegou como número continua sendo o teste da nota irmã: somar uma coluna.

## Conexões
- Princípio: [[A regra mora fora da porta que a chama]] (o formato é regra do exportador; a tela é só a porta que entrega os dados)
- Irmã: [[CSV que abre no Excel pt-BR usa ponto e vírgula, BOM e vírgula decimal]] · [[Ponto em preço brasileiro é ambíguo, e quem desempata é a contagem de casas]]
- Visto em: [[NaveX]] (o `MenuExportar` de todas as telas)
- Mapa: [[Frontend]]
