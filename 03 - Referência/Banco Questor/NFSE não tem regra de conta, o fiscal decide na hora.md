---
tags: [tipo/atomica, camada/referencia, banco/questor, contabil]
criado: 2026-07-21
---

# NFSE não tem regra de conta, o fiscal decide na hora

> **Corrigida em ago/2026** — a premissa original ("a NFSE não tem CFOP") era falsa; o título fica só para não quebrar links. A NFSE **tem** natureza (código interno `8xxxxxx` na tabela `cfop`) e **tem** tabela de contabilização como qualquer CFOP. O que ela não tem é uma regra *confiável*: parte das naturezas aponta para conta aposentada, e as genéricas recebem serviço de todo tipo, com a conta decidida nota a nota. Ver "O que se sabe hoje", abaixo.

## O que se sabe hoje (ago/2026)

A natureza de serviço é um código `8xxxxxx` em `cfop` — a descrição costuma até
carregar a conta ("Serviços Tomados S/ Retenção - 42748 ASSISTENCIA TECNICA
INFORMATICA"). Dali sai `codigotabctbfisvlrcontabil` → `tabelactbfislctoctb`
(`seq`, `naturlctoctb` +1/−1, `contactb`, `regravalorlctoctb`), igual à
mercadoria. A nota ainda carimba a tabela que usou, em
`lctofisentcfop.codigotabctbfis` — e ela bate com a config, então não há
"tabela alternativa" escondida.

O problema é que essa tabela **envelhece**. Medido em mai–jul/2026, 11.866 NFSE
de entrada com natureza única:

| | |
|---|---|
| conta da tabela acerta a conta real | 62% |
| conta habitual da natureza acerta | 86% |
| pares (empresa, filial, natureza) com TODAS as notas numa conta só, ≠ da tabela | 443, em 169 empresas |
| desses, com a conta da tabela sem nenhum movimento no trimestre | 79% |

São **dois caminhos**, medidos nos 446 pares sistemáticos do trimestre:

- **373 — natureza específica apontando pra conta aposentada.** Empresa 42,
  natureza 8001010 "Manutenção de Veículos": a tabela manda `4507 Manutenção de
  Veículos` (cadastrada em 2007) e o contábil lança em `5973 Manutenção de
  veículos` (cadastrada em mai/2024, mesmo apelido `MDV`). Conta duplicada; só o
  contábil mudou de lado.
- **73 — natureza genérica na nota.** O e-Doc importa a nota como "Serviço
  Tomados Geral" (`8000001`), cuja tabela é vala-comum (`3171 Serviços de
  Terceiros`), mas a empresa tem um **catálogo inteiro** de naturezas de
  serviço, uma por conta (`8001001` Honorários Contábeis → 4538, `8001004`
  Internet e Software → 4941, `8001015` Serv. Profiss. → 4537…), e a
  contabilização usa a específica. Empresa 767, jul/2026: as 50 NFSE entram na
  genérica e saem em 4537, 4941, 4481, 4084, 4337, 4542 — exatamente as contas
  das naturezas específicas do catálogo dela.

**Essa escolha não fica gravada na nota.** Procurado exaustivamente: a conta
(4537) não aparece em nenhuma coluna numérica de nenhuma tabela `lctofisent*`
daquela nota, nem o número da tabela (772/765); e no banco inteiro da empresa a
conta só existe em `tabelactbfislctoctb` (as tabelas 765 e 772) e em
`relacplanoecd`. O lançamento sai com o histórico `227` — o mesmo da linha da
tabela — e com a fórmula `vlrContISS`; só a conta difere. O rastro de qual
natureza foi aplicada não existe no dado.

`relacserviconatureza` (serviço municipal + fornecedor + produto → natureza) é a
candidata óbvia e **não é a resposta**: onde ela aponta para naturezas
específicas, ela acertou 0 de 5 notas contra 5 de 5 da natureza carimbada na
nota. Ela é de-para de importação do e-Doc, não regra de conta.

E há o segundo caso, esse sim o do título: natureza **genérica** (ex.: "Serviços
Tomados S/ Retenção – Serv. Profiss."), em que a conta muda de nota para nota de
propósito e nenhuma domina — 3.424 notas no trimestre.

O tratamento dos dois está em
[[Conta da natureza de serviço vem do hábito, não da tabela do ERP]].

## Consequência prática

Não existe "hipotético correto" pra calcular na NFSE — a conta que o fiscal escolheu **é** a decisão. Aprender `fornecedor → conta` do histórico erraria (validado: só ~82% dos pares empresa+fornecedor usam uma única conta; ~18% variam).

Por isso, num **balancete fiscal** (reproduzir onde a nota deveria cair), a NFSE **lançada espelha o próprio real** — entra no balancete pra ficar completo, mas **não gera divergência**, porque não há regra pra cobrar. A detecção de "conta errada" só vale onde há regra (mercadoria/CFOP). Ver [[Balancete é movimento do período, saldo é consequência]].

O que **dá** pra checar na NFSE é **inconsistência**: o mesmo fornecedor caindo em contas diferentes ao longo do tempo é candidato a erro de classificação — mas isso é auditoria de anomalia, não regra.

## Refino (jul/2026): a NFSE não lançada some do balancete — e não deveria

A NFSE **lançada** espelha; a **não lançada** cai num vão. Sem CFOP, o motor não a
reproduz; sem lançamento, não há real pra espelhar. Então uma NFSE que **precisa**
ser contabilizada (toda NFSE precisa) mas **não foi** ficava invisível — esperado
batia com real, o balancete dizia "ok" e escondia o lançamento faltando.

Conserto validado: NFSE sem lançamento é pendência certa, então prevê-se a conta
pela **moda do histórico do próprio fornecedor** e soma-se o valor ao esperado,
criando a divergência. Como só se aplica a nota **sem real**, não há falso positivo
(não sobrepõe lançamento nenhum).

Medição que sustenta (NFSE entrada 2025, conta de débito principal, 7.049 pares
empresa+fornecedor com ≥2 notas): a conta **analítica** exata só é estável em
**57%** — pior que o "~82%" citado acima, que era medida mais frouxa. Mas **um
fornecedor específico costuma ser 100%** (ex.: honorários da NAVECON, sempre na
mesma conta), e onde varia a **sintética** segura: **88%** no nível 2, **82%** no
nível 3 — é a "conta sintética que abrange serviço" como rede. Sem histórico
nenhum, a nota fica só num painel de pendentes, sem chutar conta.

Ou seja: "NFSE espelha e não gera divergência" segue válido **para a nota
lançada**; a não lançada agora aparece como o que é — uma falta.

## Conexões
- Regra e mecanismo: [[Conta da natureza de serviço vem do hábito, não da tabela do ERP]]
- Vínculo nota↔lançamento: [[Vínculo nota fiscal e lançamento contábil no Questor]]
- Visto em: [[Navetech Hub]] (Balancete Fiscal, NFSE-espelho e pendentes)
- Mapa: [[Banco Questor]]
