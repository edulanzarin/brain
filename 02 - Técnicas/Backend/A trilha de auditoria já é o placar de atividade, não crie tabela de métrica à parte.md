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

## Ao ranquear GENTE, separe produzir de consultar

Contar ação por período é uma coisa; ordenar **pessoas** pela trilha é outra, e a
segunda tem uma armadilha própria. A trilha mistura, no mesmo `count(*)`, o gesto
que **deixou algo pronto** (gerou o arquivo, gravou a regra, resolveu a
pendência) com o gesto que só **puxou dado** (consultou, abriu o detalhe,
exportou). Somados, quem mais exportou planilha sobe no ranking como se tivesse
produzido mais que quem conciliou o mês inteiro.

A correção é o catálogo de verbos carregar um `tipo` (`producao` | `leitura`), e
o painel:

- **ordenar por produção**, com o total só como desempate;
- **contar as duas** em cartões separados, porque consulta também é informação
  (é adoção, é custo de varredura) — só não é produção;
- **comparar produção com produção** no delta do período anterior: um `count(*)
  filter (where acao = any($verbosDeProducao))`, não o total de antes contra a
  produção de agora.

Corolário honesto: num módulo **somente leitura**, onde nenhum gesto grava nada,
a produção é zero e o painel tem de dizer isso — o cartão sempre zerado sai da
tela, e a frase explica que o trabalho daquele time está em outra aba. Vender
consulta como produção é o mesmo erro, só que na direção do marketing.

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
- Completa: [[Instrumentar o gesto no funil, e um teste cobra a classe de cada verbo]] — a trilha só é placar do que foi instrumentado
- Irmã: [[Ordene pela grandeza que decide, não pela que impressiona]] — produção decide, volume total impressiona
- Visto em: [[Navetech Hub]] (Contábil → Painel; e a aba No Nexo das Produtividades, onde a trilha vira ranking por pessoa)
- Mapa: [[Backend]]
