---
tags: [tipo/atomica, camada/referencia, dev/backend, banco/questor, contabil, sql]
criado: 2026-07-28
---

# Plano de contas padrão do Questor e leitura do balancete

> O plano de contas do escritório (`planoespec`, por empresa) segue uma numeração de `classifconta` estável em ~1310 empresas. Conhecer o mapa das classes e três armadilhas de leitura (resultado acumula o ano, PL pré-apuração é residual, contra-parte não é anomalia) evita construir relatório contábil errado sobre saldo bruto.

## O mapa das classes (grau-1 e grau-2 do `classifconta`)

Validado contando `planoespec` em todas as empresas (a descrição da raiz é a mesma em ~1310 delas):

| Classe | O que é | Subgrupos que importam |
|---|---|---|
| **1** | Ativo (natureza devedora) | `1.1` Circulante · `1.2` Não Circulante · **`1.4` Compensação** (memorando) |
| **2** | Passivo + PL (natureza credora) | `2.1` Circulante · `2.2` Não Circulante · **`2.4`/`2.5`/`2.6` Patrimônio Líquido** (Líquido / Social / Resultado do Exercício) · **`2.9` Compensação** |
| **3** e **4** | Receitas | classe 4 domina; a 3 é quase inexistente |
| **5** | Custos e despesas | |
| **6** | Impostos sobre o lucro (IRPJ/CSLL) | entra no resultado líquido, separado das despesas operacionais |
| **7** e **8** | Resultado / apuração (transitórias) | zeram no encerramento — ficar **fora** dos totais pra não duplicar |

Dentro de `1.1` (Ativo Circulante), subgrupos estáveis úteis pra liquidez: **`1.1.01` Disponível** (caixa/bancos), **`1.1.06` Aplicações financeiras**, **`1.1.08` Estoques**.

`natursaldo` é por conta (`1` devedora, `-1` credora) e cobre as **redutoras** no meio dos grupos (depreciação acumulada credora dentro do ativo, deduções da receita devedoras dentro da classe 4). Confie nele por conta, não no grupo.

## Três armadilhas de leitura

1. **Conta de resultado acumula o ano (year-to-date).** O `saldoFinal` de uma conta de classe 4/5/6 é o acumulado do exercício, não do mês. Ler o saldo mês a mês mostra uma reta sempre subindo (parece receita crescendo todo mês). Para desempenho do período, some o **movimento** (débito/crédito) dos meses, não o saldo — ver [[Balancete é movimento do período, saldo é consequência]]. Patrimonial (1/2) é estoque e usa saldo; resultado (4/5/6) é fluxo e usa movimento.

2. **Num balancete mensal pré-apuração, o PL registrado não inclui o resultado do exercício.** As contas de PL (`2.4`/`2.5`/`2.6`) só recebem o lucro/prejuízo no encerramento. Antes disso o resultado mora nas contas 4/5/6, então **Ativo ≠ Passivo + PL registrado** — a diferença é exatamente o resultado ainda não transportado. O balancete continua fechando: `Σ saldo (deb−cred) de TODAS as analíticas ≈ 0` (só fecha porque inclui as contas de resultado). O **PL econômico** que faz o balanço bater é o **residual: `PL = Ativo − Passivo exigível`** (`exigível` = 2.1 + 2.2, sem PL e sem compensação). Ele embute o resultado e é o PL certo pros indicadores de endividamento/imobilização.

3. **Contra-parte com saldo de sinal atípico não é anomalia.** No Questor cada cliente/fornecedor/sócio é uma **conta própria** sob o **mesmo `classifconta`** (ver [[Vínculo nota fiscal e lançamento contábil no Questor]]). Um fornecedor devedor = adiantamento/nota de crédito/pagamento a maior — normal. "Muitas irmãs sob o mesmo classif" (ex.: ≥ 3) identifica conta de terceiro: colapse num resumo, não liste uma a uma. O sinal invertido só é alarme em conta **impossível** (caixa/banco/aplicação/estoque negativo) ou **estrutural única** (Capital devedor, tributo a recuperar credor).

## Compensação e escrituração

- **Contas de compensação** (`1.4`, `2.9`, ou descrição "compensação") são memorando: inflam os dois lados igual e batem entre si. Fora de todo total e do fechamento.
- Filtrar **`tipolancamento = 'LN'`** (normal/societário) — domina de longe (LF/LS são raríssimos aqui). Ver [[Módulo contábil do Questor]].

## Por que importa

Base de qualquer análise de balancete, DRE ou balanço no [[Navetech Hub]] e em automações futuras sobre o contábil do Questor. Errar a classe ou ler saldo onde é fluxo produz relatório contábil silenciosamente errado.

## Conexões
- Princípio de leitura: [[Balancete é movimento do período, saldo é consequência]]
- Depende de: [[Módulo contábil do Questor]]
- Irmã: [[Vínculo nota fiscal e lançamento contábil no Questor]]
- Conexão read-only: [[Questor - conexão read-only e regras]]
- Visto em: [[Navetech Hub]] (Análise de Balancete)
- Mapa: [[Banco Questor]]
