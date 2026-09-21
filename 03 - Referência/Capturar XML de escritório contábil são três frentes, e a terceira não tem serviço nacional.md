---
tags: [tipo/atomica, camada/referencia, dev/backend, fiscal]
criado: 2026-09-21
---

# Capturar XML de escritório contábil são três frentes, e a terceira não tem serviço nacional

> "Baixar os XML dos clientes" soa como um downloader. São três integrações diferentes, com filas, cursores e até órgãos distintos — e a que toca MAIS clientes é justamente a que não tem webservice nacional que resolva.

## O que a carteira usa de verdade

Medido no Questor, julho/2026 (escritório inteiro). Volume manda numa coluna,
**cobertura de clientes manda em outra** — e elas discordam:

| Documento | Fluxo | Notas | Empresas |
|---|---|---|---|
| NF-e (55) | saída | 437.026 | 273 |
| CT-e (57) | entrada | 244.361 | 156 |
| NFC-e (65) | saída | 121.505 | 66 |
| NF-e (55) | entrada | 93.424 | 284 |
| NFS-e | saída | 28.034 | 159 |
| **NFS-e** | **entrada** | **9.939** | **535** |
| CT-e (57) | saída | 4.942 | 12 |

NFS-e de entrada é o menor volume entre os grandes e **o que mais empresas
toca**: quase toda empresa da carteira recebe nota de serviço. Priorizar por
volume coloca CT-e na frente; priorizar por "quantos clientes ficam sem"
coloca NFS-e.

## Frente 1 — NF-e e NFC-e

`NFeDistribuicaoDFe`, nacional, autenticado por mTLS com o certificado do
contribuinte. Fila sequencial com cursor por CNPJ — ver
[[Distribuição DFe da SEFAZ tem uma fila por CNPJ, e o segundo consumidor derruba o primeiro]].
Resolvido, desde que só um sistema consuma cada CNPJ.

## Frente 2 — CT-e

Serviço **separado** (`CTeDistribuicaoDFe`), com fila e cursor próprios. Quem
implementa só a frente 1 perde 70% das entradas do escritório. Mesma mecânica,
código a mais — é trabalho, não risco.

## Frente 3 — NFS-e, a que não fecha

Imposto municipal: cada prefeitura tem o seu sistema. O **ADN NFS-e** (Ambiente
de Dados Nacional, da Receita) publica API de DF-e onde o contribuinte consulta
documentos em que é emitente, tomador ou intermediário — mas **só alcança
municípios conveniados** ao Sistema Nacional. Município fora do convênio
continua com portal próprio, cada um com seu padrão.

E aqui as duas pontas têm tamanhos diferentes:

- **NFS-e que o cliente EMITE**: o município é o dele. A carteira está em 184
  municípios, mas concentrada — Brusque (393 empresas), Itajaí (114),
  Navegantes (110), Joinville (60), Balneário Camboriú (57) e mais cinco somam
  dois terços. Atacar por município, do maior para o menor, é viável.
- **NFS-e que o cliente RECEBE**: o município é o do prestador, e num mês só os
  prestadores vieram de **310 municípios diferentes**. Integrar prefeitura por
  prefeitura aqui não termina nunca. Ou o ADN cobre, ou essa ponta continua
  dependendo de serviço de mercado.

## Por que importa

É a diferença entre "trocar um fornecedor por um script" e "assumir três
integrações, uma delas sem solução fechada". Um serviço de captura de mercado
(SIEG e parecidos) cobra justamente por manter as três de pé — e a conta de
trazer para dentro se faz com esta tabela na frente, não com a intuição de que
é só baixar XML.

## Conexões
- Depende de: [[Distribuição DFe da SEFAZ tem uma fila por CNPJ, e o segundo consumidor derruba o primeiro]]
- Ver também: [[Modelo de dados fiscais do Questor]] · [[NFSE não tem regra de conta, o fiscal decide na hora]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Backend]]
