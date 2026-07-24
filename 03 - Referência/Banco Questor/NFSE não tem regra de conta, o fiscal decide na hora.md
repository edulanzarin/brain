---
tags: [tipo/atomica, camada/referencia, banco/questor, contabil]
criado: 2026-07-21
---

# NFSE não tem regra de conta, o fiscal decide na hora

> Diferente da mercadoria (onde o CFOP determina a conta contábil pela regra), a **NFSE de serviço não tem CFOP** e a conta de despesa é escolhida **manualmente pelo fiscal, nota a nota**. Não vem de tabela: mesmo fornecedor pode cair em contas diferentes em meses diferentes, dependendo do serviço/produto.

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
- Vínculo nota↔lançamento: [[Vínculo nota fiscal e lançamento contábil no Questor]]
- Visto em: [[Navetech Hub]] (Balancete Fiscal, NFSE-espelho e pendentes)
- Mapa: [[Banco Questor]]
