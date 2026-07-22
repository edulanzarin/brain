---
tags: [tipo/atomica, camada/referencia, banco/questor, contabil]
criado: 2026-07-21
---

# NFSE não tem regra de conta, o fiscal decide na hora

> Diferente da mercadoria (onde o CFOP determina a conta contábil pela regra), a **NFSE de serviço não tem CFOP** e a conta de despesa é escolhida **manualmente pelo fiscal, nota a nota**. Não vem de tabela: mesmo fornecedor pode cair em contas diferentes em meses diferentes, dependendo do serviço/produto.

## Consequência prática

Não existe "hipotético correto" pra calcular na NFSE — a conta que o fiscal escolheu **é** a decisão. Aprender `fornecedor → conta` do histórico erraria (validado: só ~82% dos pares empresa+fornecedor usam uma única conta; ~18% variam).

Por isso, num **balancete fiscal** (reproduzir onde a nota deveria cair), a NFSE **espelha o próprio real** — entra no balancete pra ficar completo, mas **não gera divergência**, porque não há regra pra cobrar. A detecção de "conta errada" só vale onde há regra (mercadoria/CFOP). Ver [[Balancete é movimento do período, saldo é consequência]].

O que **dá** pra checar na NFSE é **inconsistência**: o mesmo fornecedor caindo em contas diferentes ao longo do tempo é candidato a erro de classificação — mas isso é auditoria de anomalia, não regra.

## Conexões
- Vínculo nota↔lançamento: [[Vínculo nota fiscal e lançamento contábil no Questor]]
- Visto em: [[Questor Hub]] (Balancete Fiscal, NFSE-espelho)
- Mapa: [[Banco Questor]]
