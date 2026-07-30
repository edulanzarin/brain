---
tags: [tipo/atomica, camada/padrao, dev/backend, banco/questor]
criado: 2026-07-30
---

# Para alimentar o ERP, gere o arquivo de importação dele

> Quando o sistema-dono do dado é de terceiro (um ERP como o Questor), a
> tentação é escrever no banco dele — um INSERT resolve. Mas o saldo, o
> lançamento, o cadastro que o ERP mostra são o topo de uma cadeia de
> invariantes que ELE garante (partida dobrada, saldo derivado, de-para pra
> Receita, lotes, histórico). A ferramenta de apoio prepara o **arquivo de
> importação** no layout do ERP e deixa ELE gravar; herda todas as guardas de
> graça e não corrompe nada.

## O problema

No Questor, "implantar um saldo" parece gravar um número em `saldoctb`. Mas
`saldoctb` é **derivado**: o saldo real nasce de `lctoctb` (partida dobrada),
que amarra em `lotectb`, `historicoctb`, `planoespec` e no `planoespecplanorefer`
(de-para pra ECD/Receita). Um INSERT em `saldoctb` sem os lançamentos que o
sustentam bate na tela e mente em tudo o resto: o razão fica vazio, a ECD não
fecha, e o primeiro recálculo do ERP sobrescreve o número plantado. O mesmo vale
pra gerar lançamento de conciliação bancária: escrever `lctoctb` na mão pula as
regras do ERP.

Some-se a isso que o banco de produção é **somente leitura** por decreto — ver
[[Questor - conexão read-only e regras]] —, então nem existe a porta do INSERT.
Isso não é obstáculo, é a pista certa: a porta de escrita do ERP é a importação
dele.

## A solução

A ferramenta é um **preparador**, não um escritor. Ela lê (read-only), casa,
valida e **emite o arquivo no layout de importação do próprio ERP**; um humano
importa e confere lá dentro. O ERP aplica seus invariantes na ingestão, como
faria num lançamento digitado.

- O layout é fixo e conhecido (no Questor, o `.nli` de importação de lançamentos:
  `C;empresa;estab;data;contaDeb;contaCred;histórico;complemento;valor`,
  delimitado por `;`, decimal com vírgula; `TIPOLANCAMENTO`/`ORIGEMDADO` já
  fixados pelo layout).
- **A saída é uma só; a entrada é que varia.** Balancetes vêm de softwares
  diferentes — por isso o parser de origem multiplica, mas desemboca num
  **formato canônico** e o gerador do arquivo é escrito uma vez. Ver a mecânica
  em [[Navetech Hub]] (Implantação de Saldos).
- O dado próprio da ferramenta (padrões, de-para aprendido) mora no banco DELA,
  nunca no do ERP.

É o mesmo princípio de [[Importação em massa passa pela API, não pelo banco]],
virado para fora: lá você entra no seu app pelo funil do app; aqui você entra no
sistema de terceiro pelo funil dele. Nos dois casos o invariante é garantido pela
estrutura de ingestão, não por uma escrita manual que a contorna.

## O que mais vale lembrar

Vale quando existe um layout de importação documentado (a maioria dos ERPs tem,
porque migração de cliente é rotina). Se não houvesse, o caminho seria a API/rotina
oficial do ERP — nunca o INSERT. O ganho colateral: como a ferramenta só prepara,
ela pode rodar sobre a base de produção sem risco, e o passo de importar continua
sendo uma decisão humana auditável.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[Importação em massa passa pela API, não pelo banco]]
- Depende de: [[Questor - conexão read-only e regras]]
- Visto em: [[Navetech Hub]] (Implantação de Saldos e Conciliação)
- Mapa: [[Backend]]
