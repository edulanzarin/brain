---
tags: [tipo/atomica, camada/referencia, dev/backend, fiscal]
criado: 2026-09-21
---

# Capturar XML de escritório contábil são três frentes, e a municipal está fechando agora

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

## Frente 3 — NFS-e, a que está fechando agora

Imposto municipal, historicamente mais de cinco mil sistemas diferentes. Isso
mudou em 2026, e quem parar de ler na frase anterior vai desenhar o sistema
errado:

- **01/01/2026** — municípios passam a ser obrigados a autorizar a emissão da
  NFS-e Nacional e, se mantiverem emissor próprio, a **compartilhar os
  documentos com o ADN** no leiaute padronizado.
- **01/09/2026** — ME e EPP do Simples Nacional emitem **exclusivamente pelo
  Emissor Nacional** (web ou API), por força da Resolução CGSN 189/2026,
  publicada em 23/04/2026.

O **ADN NFS-e** (Ambiente de Dados Nacional, da Receita) publica API de DF-e em
que o contribuinte consulta os documentos onde é emitente, tomador ou
intermediário. Com o compartilhamento virando obrigação do município, a
promessa é justamente a ponta que parecia impossível: o tomador consultando num
lugar só o que recebeu de qualquer município.

**A ressalva que separa a lei do sistema**: obrigação publicada não é
implementação em pé nos cinco mil municípios, e a cobertura real do
compartilhamento precisa ser medida, não suposta. Pior: **o Questor não guarda
chave de NFS-e** — em jun–ago/2026, 99,99% das NFS-e de saída estão sem chave
nenhuma —, então a conferência não sai do banco. Tem de sair do próprio ADN.

E as duas pontas ainda têm tamanhos diferentes enquanto a cobertura não fecha:

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
