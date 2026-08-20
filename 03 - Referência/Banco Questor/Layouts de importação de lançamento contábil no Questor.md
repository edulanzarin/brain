---
tags: [tipo/atomica, camada/referencia, dev/backend, banco/questor]
criado: 2026-08-20
---

# Layouts de importação de lançamento contábil no Questor

> O Questor aceita mais de um layout para o mesmo lançamento contábil, e eles não são variações cosméticas: mudam separador, formato de data e separador decimal. O `.nli` é o layout descrito em arquivo (`TnItemImportacao`); o CSV é a forma curta, sem cabeçalho, que a contabilidade usa no dia a dia. Escolher o layout é escolher três formatações de uma vez.

Os dois entram pela mesma porta — a rotina de importação do Questor, nunca um INSERT: [[Para alimentar o ERP, gere o arquivo de importação dele]].

## O `.nli` ("Layout Importação Lançamento Contábeis")

Uma partida por linha, prefixo `C;`, separador `;`, data `DD/MM/AAAA`, decimal com **vírgula**:

```
C;empresa;estab;DD/MM/AAAA;contaDeb;contaCred;codHistorico;complemento;valor
```

`TIPOLANCAMENTO='LN'` e `ORIGEMDADO='3'` são valor fixo do próprio layout — não vão no arquivo. `COMPLHIST` tem **300 caracteres**; o delimitador de texto é `"`. O arquivo `.nli` que descreve isso é o próprio layout cadastrado no ERP, então dá para ler os campos e as posições direto dele.

## O CSV curto

Sem cabeçalho, separador `,`, data **`DDMMAAAA` sem barras**, decimal com **ponto**:

```
filial,DDMMAAAA,contaDeb,contaCred,valor,codHistorico,"complemento"
1,01012021,4537,1496,516.15,0," - 202019   APTA REPRESENTACOES COMERCIAIS LTDA"
```

Duas diferenças que enganam quem vem do `.nli`:

- **Não tem coluna de empresa.** O primeiro campo é a **filial** (estabelecimento); a empresa é a que está aberta na importação.
- **O código do histórico vai `0`** quando o histórico é o texto livre do último campo. O código só é preenchido para apontar um histórico cadastrado — e aí o texto vira complemento dele.

O complemento sai sempre entre aspas: descrição de extrato vem cheia de vírgula, e ele é o último campo da linha.

## Por que isso importa na hora de gerar

O mesmo dado gera arquivos diferentes por detalhe de formatação, então **o formatador é do layout, não do domínio**: data, valor e complemento se formatam em três funções por layout, e o gerador só compõe. Foi o que permitiu a Conciliação do [[Navetech Hub]] trocar `.nli` por CSV mexendo só na camada de formatação, com a Implantação seguindo no `.nli` intocada.

## Conexões
- Índice: [[Banco Questor]] · Contábil: [[Módulo contábil do Questor]]
- Depende de: [[Questor - conexão read-only e regras]]
- Irmã: [[Contas bancárias e layout de contabilização no Questor]]
- Princípio de fundo: [[Para alimentar o ERP, gere o arquivo de importação dele]]
- Visto em: [[Navetech Hub]] (Conciliação e Implantação de Saldos)
- Mapa: [[Banco Questor]]
