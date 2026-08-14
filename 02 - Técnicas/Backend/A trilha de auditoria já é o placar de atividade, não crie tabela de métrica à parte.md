---
tags: [tipo/atomica, camada/tecnica, dev/backend]
criado: 2026-08-14
---

# A trilha de auditoria já é o placar de atividade, não crie tabela de métrica à parte

> Se o sistema já registra "quem fez o quê" numa trilha de auditoria (verbo da ação + timestamp + um `jsonb` de detalhe), esse mesmo log é a fonte pronta do painel de atividade: `count(*) filter (where acao = ...)` por período dá "quantas vezes rodou", e uma métrica dentro do `detalhe` (ex.: `linhas`) somada dá "quantos dados". Não se cria uma tabela de contador separada.

## A regra

Toda ação que vale contar no painel (gerar arquivo, rodar conferência, exportar,
triar) já deveria estar sendo **auditada** — então o painel lê a trilha, não um
placar paralelo. Duas colunas fazem o trabalho:

- **`acao`** (verbo estável, ex.: `contabil.conciliacao.gerar`) → o QUE, por
  `count(*) filter (where acao = '...')`.
- **`detalhe` jsonb** → a MÉTRICA daquele evento (ex.: `{linhas: 340, total: ...}`)
  → `sum((detalhe->>'linhas')::int)` dá o volume acumulado.

O `criado_em` recorta o período; o `codigoempresa` (quando existe) aplica o
escopo; o `modulo` filtra o painel do módulo certo.

## Por que

Uma tabela de métrica separada é **duas fontes da mesma verdade** que divergem:
alguém registra a ação mas esquece de incrementar o contador, ou vice-versa. A
trilha já é escrita no caminho da ação (é requisito de auditoria), então contar a
partir dela é sempre consistente com o que de fato aconteceu — e de graça. Bônus:
o mesmo evento serve investigação (a tela de auditoria) e placar (o painel).

Casa com [[A definição em dado dirige o comportamento, não um caso no código]]:
o número no painel é derivado do log, não mantido à mão.

## Na prática

- Ao somar uma métrica do `detalhe`, **case só a ação certa** para não castar
  `jsonb` de eventos que não a têm: `sum(case when acao = 'x.gerar' then
  (detalhe->>'linhas')::int end)`, não `(detalhe->>'linhas')::int` cru sobre tudo.
- A trilha é **best-effort** (não derruba o fluxo se falhar) — então o placar é um
  retrato "do que registrou", bom o bastante para gerência, não contabilidade
  fiscal. Se algum evento precisar ser garantido, aí sim ele vira dado de
  primeira classe.
- Densifique a série no tempo com `generate_series` (meses sem ação viram zero,
  não sumem).

## Conexões
- Serve a home-placar de: [[A home de um módulo é o resumo que carrega sozinho; automação não abre sozinha]]
- Visto em: [[Navetech Hub]] (Contábil → Painel)
- Mapa: [[Backend]]
