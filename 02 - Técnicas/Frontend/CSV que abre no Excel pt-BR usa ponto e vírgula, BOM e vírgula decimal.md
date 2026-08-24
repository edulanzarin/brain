---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-08-24
---

# CSV que abre no Excel pt-BR usa ponto e vírgula, BOM e vírgula decimal

> Exportação "funciona" (o arquivo baixa, o Excel abre) e mesmo assim está quebrada: sem BOM os acentos viram lixo, sem `;` tudo cai numa coluna só, e sem vírgula decimal a coluna de valor chega como TEXTO e não soma.

## O problema

O CSV padrão (vírgula separando campos, ponto decimal, UTF-8 sem BOM) é o formato errado para quem vai abrir no Excel em português com duplo clique. Os três sintomas aparecem separados, e o terceiro é o pior porque não parece defeito: a planilha abre bonita, as colunas estão certas, e a soma da coluna de valor dá zero — o Excel leu `9540484544.72` como texto, já que naquele locale o ponto é separador de milhar.

## A solução

Três detalhes, todos no gerador:

```ts
// separador: ponto e vírgula (o Excel pt-BR espera o do locale)
const corpo = [cabecalhos, ...linhas].map((l) => l.map(esc).join(";")).join("\r\n");
// BOM UTF-8 na frente: sem ele o Excel assume a página de código do sistema
const blob = new Blob(["﻿" + corpo], { type: "text/csv;charset=utf-8" });

// decimal com vírgula — só nos números fracionários; inteiro pode ir cru
const decimalBR = (v: number) => v.toFixed(2).replace(".", ",");
```

Escapar campo que contenha `"`, `;` ou quebra de linha entre aspas duplas (dobrando as aspas internas) fecha o resto.

## O que mais vale lembrar

- **Inteiro não precisa de tratamento**; só o fracionário. Formatar quantidade com separador de milhar (`1.234`) faz o inverso do que se quer: aí sim vira texto.
- Data também é locale: `31/07/2026` entra como data; `2026-07-31` costuma virar texto.
- A conferência é abrir o arquivo e **somar uma coluna** — se o total sai zero ou o Excel alinha os números à esquerda, chegou como texto.
- Quem exporta dado fiscal ou pessoal registra a exportação na trilha: [[A trilha de auditoria já é o placar de atividade, não crie tabela de métrica à parte]].

## Conexões
- Princípio: (folha isolada — nenhum princípio da Base cobre; é compatibilidade de formato)
- Visto em: [[Navetech Hub]]
- Mapa: [[Frontend]]
