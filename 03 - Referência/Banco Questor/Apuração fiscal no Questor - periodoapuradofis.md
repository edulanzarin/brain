---
tags: [tipo/atomica, camada/referencia, dev/backend, banco/questor]
criado: 2026-08-31
---

# Apuração fiscal no Questor - periodoapuradofis

> O FECHAMENTO mensal do fiscal — quem apurou qual imposto de qual empresa, em que competência e quando — mora em `periodoapuradofis` (64 mil linhas) e, para as retenções, em `periodoapuradofisretido` (15 mil). As duas têm `codigousuario` + `datahorausuario`, e **zero linhas do usuário 0**: é trabalho humano de ponta a ponta, o oposto do fiscal de nota, onde a integração faz 98%.

## Colunas

`codigoempresa`, `codigoestab`, `tipoimposto`, `tipoapuracao`, `datainicial`,
`datafinal` (a competência apurada), `codigousuario`, `datahorausuario` (o
carimbo do trabalho), `atualizado`, `memcalculo` (text, **vem vazio** nesta
base). `periodoapuradofisretido` tem a mesma forma, sem `memcalculo`.

Índice em `(codigoempresa, codigoestab, tipoimposto, datainicial)` — **não** em
`datahorausuario`. Filtrar por período de trabalho varre a tabela, o que a 64
mil linhas é irrelevante (0,76 s para o mês do escritório inteiro, medido).

## Uma apuração são VÁRIAS linhas

Um fechamento grava uma linha **por imposto**: em ago/2026 foram 2.227 linhas
para **968 fechamentos** distintos por `(codigoempresa, codigoestab,
competência)`. A assinatura do lote é visível a olho nu — os tipos 1, 13 e 77
saem sempre com contagem idêntica (383 cada, nas mesmas 228 empresas).

Contar linha infla o trabalho em 2,3× e desequilibra qualquer comparação entre
pessoas. Ver [[A unidade de contagem é o ato, não a linha que ele deixou]].

## `tipoimposto` NÃO tem tabela de rótulo

Verificado: das 58 tabelas do banco que têm a coluna `tipoimposto`, nenhuma é um
cadastro de nome. A tabela `imposto` existe e é de **outro domínio** — lá o
código 77 é "Dívida Ativa Ajuizada parcelamento", e aqui o 77 sai junto do ICMS
todo mês. O `memcalculo`, que poderia nomear, está vazio.

Dois códigos foram **provados** por impressão digital em `lctofissaicfop` (mesma
coluna, mesmo banco), sobre jul/2026:

- **1 = ICMS** — alíquotas 12, 17, 7, 4, 18, 25 e 19,5% (interestadual 12/7/4,
  interno 17/18/19,5, seletivo 25) e aparece só em NFE, NFCE e CTE.
- **2 = ISS** — alíquotas 2 e 3%, e aparece **exclusivamente** em NFSE. Era o
  candidato natural a IPI pela ordem numérica, e não é: a alíquota e a espécie
  desmentem. (Fica o aviso — a ordem dos códigos não segue a ordem "óbvia" dos
  tributos.)

O alcance na apuração é coerente: 364 empresas apuram o tipo 1 e 106 o tipo 2.

Os demais (10, 13, 17, 29, 33, 45, 71, 76, 77 e as retenções 20, 27, 51, 53)
seguem **sem nome**, e o [[Navetech Hub]] os mostra pelo código de propósito
(ver [[Rótulo feito de chave técnica aponta para o registro errado quando os dois ids se parecem]]).
Quem do time fiscal souber, nomeia.

## O atraso de apuração é o indicador

`datahorausuario::date - datafinal` é a distância entre o fim da competência e o
fechamento, e ela varia MUITO por imposto (medido em jul-ago/2026, mediana e
p90):

| tipo | linhas | mediana | p90 | empresas |
|---|---|---|---|---|
| 71 | 1.751 | **293** | 485 | 37 |
| 2 (ISS) | 126 | 141 | 424 | 13 |
| 76 | 847 | 25 | 212 | 297 |
| 33 | 550 | 18 | 24 | 251 |
| 1 (ICMS) / 13 / 77 | 755 cada | 15 | 44 | 228 |

Duas armadilhas ao ler isso:

- **Existe apuração ANTECIPADA**: o mínimo medido é −174 dias (fechar antes de a
  competência terminar), e há período com `datafinal` no futuro (7 linhas de
  1.751 no tipo 71). A escada precisa de piso aberto para baixo.
- **Média mente feio** por causa da cauda: use mediana e p90.

Todos os períodos são MENSAIS (`datafinal - datainicial + 1` ≈ 31 em todos os 13
tipos).

## O que NÃO usar

`impostocontrole` (chave, valor, vencimento e status do imposto) parece a tabela
perfeita para "guias a pagar" e tem **4 linhas, a mais recente de 2008** — o
módulo nunca foi usado nesta base. `impressaorelat` (quem imprimiu qual
relatório) tem 181 linhas em dois meses: fino demais.

## Conexões
- Ver também: [[Modelo de dados fiscais do Questor]] · [[Impostos no Questor - onde fica cada um]]
- Doutrina de contagem: [[A unidade de contagem é o ato, não a linha que ele deixou]]
- Recorte de produtividade: [[Produtividade se mede pela hora do registro, não pela data do fato]]
- Visto em: [[Navetech Hub]] — aba Apuração da Produtividade do Fiscal
- Mapa: [[Banco Questor]]
