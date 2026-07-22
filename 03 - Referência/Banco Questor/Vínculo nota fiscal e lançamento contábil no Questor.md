---
tags: [tipo/atomica, camada/referencia, dev/backend, banco/questor, sql]
criado: 2026-07-19
---

# Vínculo nota fiscal e lançamento contábil no Questor

> A contabilização de uma nota fiscal são lançamentos em `lctoctb` com `codigooriglctoctb='FI'` e `chaveorigem = 'ME'/'MS' + a chave da nota` (com zero-fill). É o elo para conferir se a nota foi contabilizada (base da automação "Conferência Fiscal").

## O elo (verificado jun/2026)

- **Entradas**: `lctoctb.chaveorigem = 'ME' || lpad(chavelctofisent::text, 10, '0')`.
- **Saídas**: `lctoctb.chaveorigem = 'MS' || lpad(chavelctofissai::text, 10, '0')`.
- Só nos lançamentos com `codigooriglctoctb='FI'`. Uma nota gera **vários** lançamentos (partida dobrada: mercadoria + cada imposto) com o **mesmo** `chaveorigem`.
- Outros prefixos de `chaveorigem` em FI: **`IM`** = impostos apurados (ex.: `IMP01`, mensal), **`RE`** = retenções. Não são notas.
- **`datalctoctb` = `datalctofis`** (100% na amostra) → dá pra filtrar os dois pelo mesmo período.
- `numerodcto`/`transctb`/`codigolctoprog` **não** servem de elo (nulos ou irrelevantes). A nota (`lctofis*`) **não tem** coluna de vínculo com o contábil — o elo mora no `chaveorigem` do lançamento.

Conferência = LEFT JOIN da nota (por `chave`) contra o conjunto `substring(chaveorigem from 3)::bigint` dos FI `ME`/`MS` da empresa no período. Nota sem match = **não contabilizada**.

## Prefixos de `chaveorigem` (origem FI não é só nota)

O prefixo de 2 letras do `chaveorigem` diz o que é o lançamento FI:
- **`ME`/`MS`** = nota individual (entrada/saída) — o elo por chave.
- **`IM`** = apuração mensal de imposto (ex.: `IMP01`): ICMS/PIS/COFINS a recolher × créditos. Não é nota. **`substring(from 3)` NÃO é numérico aqui** (`IMP01`→`P01`) — cast pra bigint estoura; filtrar `~ '^M[ES][0-9]+$'` antes.
- **`RE`** = retenção.
- **`MOV`** = **consolidação** (ex.: `MOVMS202605000001`): venda de varejo/cupom (ECF) agregada do mês num lançamento só. O histórico rotula UMA NF de referência, mas **não é** a nota individual (visto: `MOVMS...` de R$ 210k com histórico "NF 4830", sendo que a nota fiscal 4830 é uma saída de R$ 379,89 — o rótulo engana).

**Armadilha grave da Conferência por nota: venda consolidada gera falso "não contabilizada" em massa.** O varejo lança as vendas em BLOCO (`MOV`), não nota a nota. Como a Conferência procura lançamento `MS` por chave, TODAS as notas consolidadas aparecem como pendentes. Validado emp 1015/mai: **108 notas** CFOP 6107007 (R$ 207,5k) com **zero** lançamento individual, mas 1 consolidação `MOVMS...` de R$ 210,3k cobrindo tudo. Não é erro do escritório. O **balancete fiscal** (que soma TODA origem FI, com MOV/IM espelhados) já pegava certo.

**Resolvido (jul/2026) — a Conferência reconhece consolidação.** Antes de dar pendência: reúne as contas que o MOV toca no período (`chaveorigem like 'MOV%'`, débito e crédito, chave "natureza:conta") e cruza com as contas fixas que a nota postaria pelo plano. O MOV é uma **partida** (débito clientes/caixa × crédito receita — na 1015: D 142 Clientes Diversos × C 2606 Vendas de Produtos a Prazo). Se a nota tem, entre suas contas fixas, um débito coberto E um crédito coberto pelo MOV, a partida principal dela coincide com a consolidação → situação **"consolidada"** (lançada em bloco), não pendente. Impostos (ICMS/IPI) ficam de fora: vão pra apuração mensal, não pro MOV. Emp sem MOV: conjunto vazio, nada muda. Resultado na 1015/mai: 108 pendentes → 0; 108 consolidadas.

**Armadilha dentro da armadilha (a que quase furou o fix):** o critério tinha que ser por **partida/conta**, não por estrutura do plano. O primeiro corte cruzava só o componente "valor contábil" do plano do Questor — e falhou silencioso, porque esses CFOPs da 1015 vinham de **override**, que colapsa tudo num componente único `id: "override"` com linhas SEM fórmula (`regra_valor` null). Detector que casa pela estrutura rica (id de componente, fórmula) funciona na fonte rica (config do Questor) e morre na fonte pobre (override). O que as duas fontes têm em comum é a **conta e a natureza** — casar por aí.

## Nem toda nota deve ser contabilizada (senão dá falso positivo)

- **NFSe, CTe, NFCOM, NF3E → 100% contabilizadas.**
- **NFe de mercadoria → parcial**: só as que afetam resultado/patrimônio. **Remessas, retornos, industrialização por encomenda, consignação, vasilhame, amostra grátis, bonificação, comodato NÃO geram lançamento** (correto, não é erro). Numa indústria isso é a maioria das NFe (ex.: empresa 1200 em jun: 951 de 1358 NFe-entrada eram remessa/retorno).
- **Canceladas** não devem ser contabilizadas.

## Como saber se um CFOP "deve contabilizar"

**Resolvido em jul/2026 — ver [[Plano de contabilização por CFOP no Questor]].** A configuração existe e é exata: as colunas `cfop.codigotabctbfis*` (na tabela `cfop`, que é **por empresa + estabelecimento**) apontam, por tributo, a tabela de contabilização. **CFOP sem nenhum slot preenchido = não contabiliza** — remessa, retorno, industrialização. Não precisa de heurística. Confirmado na 1200: o CFOP `1901002` ("Entrada para Industrialização por Encomenda") vem com zero componentes, ou seja, o próprio ERP diz que não gera lançamento.

O que eu tinha escrito antes, corrigido:

- `cfoptabctbfis` **não** é o caminho — 7 linhas no banco inteiro (não é "vazia só na 1200"). `cfop.finalidade` zerada segue valendo.
- A **regra empírica** ("CFOP é contabilizável se ≥1 nota dele foi contabilizada no período") funcionava como aproximação — 951 falsos → 37 gaps reais nas entradas da 1200 — mas foi **substituída** pela config real. Continua útil para outra coisa: calibrar *quais contas recebem lançamento nota a nota*, já que ICMS/IPI de saída são contabilizados na apuração mensal (prefixo `IM`), não na nota.

## Contabilização em duplicidade (contabilizou a mesma nota 2×)

**Sinal limpo (validado jul/2026):** a MESMA partida `(contactbdeb, contactbcred, valorlctoctb)` da nota aparece em **2+ dias distintos** de `datahoralctoctb` → re-rodaram a contabilização. Os dois lançamentos compartilham o `datalctoctb` (competência = data da nota), então o filtro por período da Conferência (que é por `datalctoctb`) captura ambos; quem denuncia é o `datahoralctoctb` (quando foi digitado).

Validado na **empresa 264**: um lote de ~18 notas foi contabilizado em 16/06 e **de novo em 23/06** (R$ 151.828,75 lançados a mais). Ex.: NF 1836, R$ 11.720,83, partida `déb 4538 / créd 5000118` idêntica nos dois dias.

Becos sem saída (não usar, já testados):
- **`codigolotectb` é NULO** nos lançamentos FI — não serve de chave de lote/batch.
- **Razão soma/valor ≈ 2 é NORMAL**, não duplicidade: cada linha é débito OU crédito, então `sum(valorlctoctb) ≈ 2 × valorcontabil` numa contabilização única (metade das notas cai em ~2). Duplicata seria ~4, que não apareceu na amostra limpa.
- **"Qualquer partida repetida" (tot > distintas) é ruidoso**: nota com vários `numerodcto` no mesmo instante repete centavos de imposto iguais entre documentos distintos.
- **"2+ sessões de horário" sozinho é ruidoso**: nota lançada em parcelas em dias diferentes tem partidas DIFERENTES (não repete) — parcela ≠ duplicata. Por isso o critério exige partida IDÊNTICA em dias distintos.

Na automação isso virou a situação **"duplicada"** da Conferência, com precedência sobre a conferência de conta (com valores dobrados, a divergência de conta seria enganosa).

## `classifconta` não é única por conta — não escope drill-down por ela

Várias contas analíticas DISTINTAS no `planoespec` dividem a **mesma**
`classifconta` — o Questor cria uma conta por pessoa (cada cliente/fornecedor) e
todas herdam a classificação do grupo (ex.: emp 1015, ~149 contas de cliente com
`classifconta='1.1.02.001.001'`). Consequência: um **drill-down ou rollup que
resolve as contas por `classifconta = X or like X.%` soma todas as irmãs**, não a
conta clicada. Regra: linha **analítica** → escopar por `contactb`; só a
**sintética** → somar a subárvore por classif. (Bug real no balancete do Questor
Hub: clicar na conta 142 somava as 149 contas de cliente.)

## Não confundir

- **`codigooriglctoctb='IP'`** (Importação) = movimento **bancário** (Pix/TED, extrato/conciliação), **não** nota fiscal. Em jun era a maior origem (87k). Ver [[Módulo contábil do Questor]] (mapa de origens).

## Conexões
- Índice: [[Banco Questor]] · Contábil: [[Módulo contábil do Questor]]
- Visto em: [[Questor Hub]] (abas Conferência Fiscal, Conferência de Contas e Configuração no módulo Contábil)
- Notas fiscais: [[Modelo de dados fiscais do Questor]] · Impostos apurados: [[Impostos no Questor - onde fica cada um]]
- Config das contas por CFOP: [[Plano de contabilização por CFOP no Questor]]
- Mapa: [[Banco Questor]]
