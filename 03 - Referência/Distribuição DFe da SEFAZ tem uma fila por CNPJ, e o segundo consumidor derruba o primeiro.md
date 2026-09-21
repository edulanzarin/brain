---
tags: [tipo/atomica, camada/referencia, dev/backend, armadilha, fiscal]
criado: 2026-09-21
---

# Distribuição DFe da SEFAZ tem uma fila por CNPJ, e o segundo consumidor derruba o primeiro

> O `NFeDistribuicaoDFe` entrega os documentos de interesse de um CNPJ em fila sequencial, controlada por um cursor (`ultNSU`) que é **do CNPJ, não do sistema**. Dois sistemas consumindo o mesmo CNPJ disputam a mesma fila, e o controle de consumo indevido bloqueia o CNPJ inteiro por uma hora — inclusive para quem estava lá antes.

## O serviço

Webservice nacional, SOAP, autenticado pelo próprio certificado (mTLS): a
requisição `distDFeInt` não vai assinada, quem prova identidade é o `.pfx` no
handshake. Três formas de pedir: `distNSU` (a fila, a partir de um cursor),
`consNSU` (um documento específico) e `consChNFe` (por chave de acesso).

- Produção: `https://www1.nfe.fazenda.gov.br/NFeDistribuicaoDFe/NFeDistribuicaoDFe.asmx`
- `cUFAutor` é o código IBGE da UF de quem consulta (SC = 42), não a sigla.
- Cada lote traz no máximo 50 documentos, cada um em `docZip` (gzip + base64).

## A armadilha do cursor

`ultNSU` parece do cliente — "de onde eu quero continuar" — e é do CNPJ. A SEFAZ
guarda até onde aquele CNPJ já foi servido, e pedir de novo do começo é
**consumo indevido**:

```
cStat 656 — Rejeicao: Consumo Indevido (Deve ser utilizado o ultNSU nas
solicitacoes subsequentes. Tente apos 1 hora)
```

A rejeição vem com o `ultNSU` real na resposta — em set/2026, consultando a
NAVECON do NSU zero, veio `000000002297208`. Dois milhões e trezentos mil
documentos já servidos: **alguém já consome essa fila há anos**. No caso, o
e-Doc do Questor, que é o que explica `origemdado = 3` dominando as notas (ver
[[Modelo de dados fiscais do Questor]]).

A consequência é operacional, não teórica: um segundo sistema que passe a
consultar o mesmo CNPJ faz o primeiro tomar 656 e ficar **uma hora sem baixar
nota**. Num escritório contábil, isso é o fiscal sem XML — e o culpado não
aparece em log nenhum do lado de quem parou.

## O que fazer quando a fila já tem dono

Não disputar. `consChNFe` busca o documento por chave de acesso e **não mexe no
cursor**, então convive com o consumidor existente. Serve quando já se sabe
quais notas se quer — e num escritório se sabe: o Questor guarda
`chavenfeent`/`chavenfesai` de tudo que foi escriturado. O preço é uma chamada
por documento, contra 50 por lote da fila.

## SOAP 1.2 ou 1.1, nunca os dois

O endpoint é ASP.NET (`.asmx`) e aceita as duas versões, mas não a mistura. Com
`Content-Type: application/soap+xml` (1.2), a action vai **dentro** do
Content-Type; o header `SOAPAction` é de SOAP 1.1. Mandando os dois, a resposta
volta **HTTP 200** com envelope de erro — sucesso para quem olha só o status.
Ver [[Contador que conta sucesso de promessa afirma que deu certo]].

## Manifestação

Pelo manual (NT 2014.002), o XML completo de uma nota de entrada só é liberado
depois que o destinatário registra ciência ou confirmação da operação. Sem isso,
a entrada chega como `resNFe` — resumo com chave, emitente, valor e data. Dá
para apontar nota não escriturada; não dá o arquivo. Manifestar é escrita na
SEFAZ em nome do contribuinte, e não se faz por conta própria.

## Conexões
- Depende de: [[Modelo de dados fiscais do Questor]]
- Ver também: [[Chamada externa tem timeout e erro tratado]] · [[Contador que conta sucesso de promessa afirma que deu certo]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Backend]]
