---
tags: [tipo/projeto, projeto/questor-bi]
criado: 2026-07-18
status: ativo
codigo_em: ~/Dev/questor-bi
---

# Questor BI

> Dashboard web de Business Intelligence sobre a base PostgreSQL do sistema contábil Questor (banco "Navecon" do escritório). Objetivo: visualizar valores, quantidades e impostos das notas fiscais com muitos filtros, e crescer para outros módulos (Contábil, Folha, Patrimônio).

Código em: `~/Dev/questor-bi`

## Estado atual

Módulo **Fiscal** funcionando com dados reais. Navegação pela **sidebar em acordeão**, com 6 seções: Painel, Análises, Tributos, Produtividade, Conformidade, Dados.

- **Painel** (`/fiscal/painel`): resumo da movimentação — KPIs (entradas, saídas, empresas, canceladas, variação vs período anterior), resumo de devoluções/cancelamentos, evolução temporal, donut por espécie e o card de impostos (ICMS/ST/IPI/ISS/PIS/COFINS + retenções + DIFAL/FCP/FUNRURAL). Fontes dos impostos: [[Impostos no Questor - onde fica cada um]].
- **Análises** (`/fiscal/analises`): rankings/distribuições — top empresas, fornecedores/clientes, produtos, CFOPs, UF, municípios e modalidade de frete.
- **Tributos** (`/fiscal/tributos`): análise tributária das saídas — KPIs (tributos destacados, **carga tributária efetiva** = impostos÷faturamento, DIFAL+FCP a recolher, ICMS), reaproveita o `ImpostosCard` (composição ICMS/ST/IPI/ISS/PIS/COFINS + retenções, com ent/saí), **DIFAL+FCP por UF de destino** (`lctofissaidifal` × `vlricmsintufdest+vlricmsfcpufdest`), **regime tributário por CST de PIS/COFINS** (`cdsituatributpis`, tributado × monofásico × alíquota zero × isento…) e **carga por empresa** em tabela (ICMS+IPI+ST+ISS ÷ faturamento; PIS/COFINS ficam de fora do por-empresa por serem tabela separada). Endpoints `/api/fiscal/tributos-{difal,cst,carga-empresas}` + reuso de `/impostos`.
- **Produtividade** (`/fiscal/produtividade`): quanto cada **colaborador** lançou (via `codigousuario` → `usuario`). KPIs (notas lançadas, colaboradores ativos, média/pessoa, % automático — sempre focados nos humanos), **ranking em tabela de largura total** (notas ent+saí, empresas atendidas, valor movimentado, canceladas; ordenável por coluna; altura limitada com scroll + cabeçalho sticky), série notas/dia (ent×saí) e um **calendário de atividade estilo GitHub** (sempre diário, um quadrado por dia, ocupando a largura toda; cobre o período selecionado — que agora é limitado a 1 ano, ver abaixo). O usuário `0` = **Sistema** (importação automática/e-Doc) **entra no ranking** como linha própria marcada "automático" (não é colaborador, mas mostra o volume que passou pelo sistema); toggles "ocultar sistema" e "só ativos". Endpoints `/api/fiscal/produtividade{,-serie,-calendario}`.
- **Conformidade** (`/fiscal/conformidade`): saúde fiscal das saídas — KPIs de pendências (itens com **NCM inválido/genérico** `9999.99.99`/`0000.00.00` + nº de produtos a corrigir; canceladas; **denegadas/inutilizadas** via `cdsituacao`≠0; NFe/NFCe/CTe **sem chave** de 44 díg, só modelos 55/65/57), distribuição de `cdsituacao` e **ranking de empresas com mais pendências**. Descartei gaps de numeração (ruído: empresa não registra a sequência toda → 400k falsos) e itens sem CFOP (=0). Endpoints `/api/fiscal/conformidade{,-empresas}`. Consultas de item (`lctofissaiproduto`) dropam `especies` e usam `incluirCanceladas` (a tabela de item não tem `especienf`/`cancelada`); filtro de NCM inválido vai no WHERE p/ o planner reduzir por produto antes (senão `count(distinct)` sobre 1,2M itens leva ~15s).
- **Dados** (`/fiscal/dados`): todas as notas em tabela paginada, com filtros (situação todas/normais/canceladas, busca por nº/contraparte, tipo) e **drill-down dos itens**. Padrão SQL em [[Receitas SQL do Questor]].

Fiscal está bem coberto (6 seções; Recebíveis foi criada e depois removida a pedido do Eduardo).

## Módulo Contábil (iniciado)

Primeira automação de fato interligando Fiscal ↔ Contábil. Módulo tem shell/filtro próprios: **uma empresa por vez + período (≤ 1 ano)** (reusa `useFiltros`/`parseFilters`; filtro em `conf-filter-bar`).

- **Conferência Fiscal** (`/contabil/conferencia`): reconciliação — quais notas fiscais **não foram contabilizadas**. Lógica em [[Vínculo nota fiscal e lançamento contábil no Questor]] (lctoctb origem FI, `chaveorigem` = `ME`/`MS` + chave da nota). KPIs por lado (ent/saí): total, contabilizadas, **pendentes** (a lançar, com valor), "não exigem lançamento" (remessas/retornos filtrados pela regra empírica de CFOP contabilizável), canceladas. Tabela das pendentes (nº, data, contraparte, CFOP, valor) com busca. Endpoint `/api/contabil/conferencia`. Validado: empresa 1200/jun → 37 pendentes reais nas entradas (compras esquecidas), 0 nas saídas.

- **Conferência de Contas** (`/contabil/contas`): o passo seguinte — entre as notas **já contabilizadas**, quais foram para a **conta contábil errada**. Compara os lançamentos FI de cada nota contra o plano do CFOP e aponta 4 tipos: conta fora do plano, lançamento faltando, valor divergente, natureza invertida (débito/crédito trocados). Endpoint `/api/contabil/divergencias`.
- **Configuração** (`/contabil/configuracao`): o plano de contabilização por CFOP, **carregado pronto do Questor** (ver [[Plano de contabilização por CFOP no Questor]]). Lista os CFOPs movimentados no período com os lançamentos esperados; editar um CFOP grava um **override** que passa a valer no lugar do plano do ERP (e a conferência cobra a versão nova); dá pra reverter pro Questor. Endpoint `/api/contabil/plano` (GET faz o merge, PUT salva, DELETE reverte).

Duas armadilhas que quase mataram a Conferência de Contas (as duas viraram regra no código, detalhe em [[Plano de contabilização por CFOP no Questor]]): (1) **ICMS e IPI de saída são contabilizados na apuração mensal**, não nota a nota — cobrá-los por nota apontava 1170 de 1186 saídas como erradas; a solução foi só cobrar conta que comprovadamente recebe lançamento nota a nota no período. (2) **Valor só é conferível em nota de CFOP único** — com vários CFOPs não dá pra atribuir a parcela de tributo de cada um. Depois disso: 7 apontamentos em 660 entradas e 2 em 1186 saídas (1200/jun), incluindo uma nota lançada em `43763` quando o plano manda `42763` (erro de dígito) e outra em `25210` ("Gás") quando o CFOP manda `42759` ("Gás Empilhadeiras").

Próximos passos possíveis no Contábil: balancete/DRE/razão (usar `saldoctb`/`lctoctb`/`planoespec`), mais conferências. Ver [[Módulo contábil do Questor]] e o mapa [[Banco Questor]].

Devoluções e cancelamentos hoje entram como **resumo** no Painel (os endpoints detalhados existem no código, se um dia virar seção própria de novo). Apuração foi tirada: era estimativa gerencial (débito−crédito), não a oficial do SPED — ver nota em [[Impostos no Questor - onde fica cada um]].

Filtros compartilhados no topo (período, empresas multi-seleção, espécie, **grupos** criados no app via localStorage) — na URL e **preservados ao navegar entre seções**. O toggle **Valor | Quantidade** só aparece onde muda algo (Painel e Análises). Tema claro/escuro.

**Período limitado a 1 ano** (`MAX_DIAS_PERIODO` em `fiscal-filters.ts`): evita consultas pesadas nas tabelas gigantes e trava. Guardado em duas camadas — `parseFilters` recusa (`400`) qualquer range > 366 dias em todos os endpoints, e o seletor de período personalizado no `filter-bar` já limita o fim a 1 ano (toast avisando). Os presets todos cabem em 1 ano.

**Padrão de navegação (reusar nos próximos módulos)**: sidebar em **acordeão** (cada módulo expande/colapsa, o ativo abre por padrão — escala pra muitos módulos/seções); módulo → seções (rotas); um `layout` do módulo segura header + filtros compartilhados; cada seção é uma página que busca só os seus dados. Config das seções em `src/lib/fiscal-secoes.ts`.

O conhecimento do banco Questor (schema, impostos, canceladas, devoluções, SQL) vive aqui no Brain em `03 - Recursos/Banco Questor` — consultar de lá para novas automações.

## Stack e contexto técnico

- Next.js 16 (App Router) + React 19, TypeScript, Tailwind v4.
- React Query pra data-fetching com `keepPreviousData` (refetch sem piscar).
- Recharts pros gráficos. Paleta seguindo [[Validar paleta de gráficos antes de escolher cores]].
- `pg` com pool, conexão **somente leitura** (`default_transaction_read_only=on`, `statement_timeout` 60s) — BI nunca altera o Questor.
- **Segundo banco, próprio e gravável** (`src/lib/app-db.ts`, pool separado): Postgres 17 em Docker Compose na porta **5433**, migrations SQL versionadas em `migrations/` com runner próprio (`npm run migrate`). Guarda os overrides do plano de contabilização e é onde vão morar login/usuários/permissões. Subir com `npm run db:up && npm run migrate`. O Questor continua intocado.
- Rodar: porta 3000 do Eduardo já é ocupada por outra app; uso `-- -p 3210`.

## Decisões importantes

- As abas de análise (Resumo/Impostos/Detalhes) são **só agregados** — o BI é sobre números. Mas o Eduardo pediu depois uma aba **Dados** com o grid bruto (nota a nota + itens) pra pesquisar tudo; convivem: agregado pra analisar, bruto pra investigar.
- Grupos de empresas vivem no app (localStorage), não no banco — ver [[grupoprocessam do Questor não é grupo de empresas]].
- Contraparte da nota = `codigopessoa` → `pessoa`, validado empiricamente — ver [[Modelo de dados fiscais do Questor]].

## Aprendizados (viram notas atômicas)

Banco Questor (pasta `03 - Recursos/Banco Questor`):
- [[Questor - conexão read-only e regras]]
- [[Modelo de dados fiscais do Questor]]
- [[Impostos no Questor - onde fica cada um]]
- [[Canceladas e devoluções no Questor]]
- [[Receitas SQL do Questor]]
- [[grupoprocessam do Questor não é grupo de empresas]]
- [[Vínculo nota fiscal e lançamento contábil no Questor]]
- [[Plano de contabilização por CFOP no Questor]]

Gerais de dev:
- [[Agregar antes de juntar em tabelas gigantes no Postgres]]
- [[router.replace do Next falha no build de produção]]
- [[Validar paleta de gráficos antes de escolher cores]]

Design (reutilizável em outros projetos — ver [[Design]]):
- [[Sistema de cores e tema do dashboard]]
- [[Padrões de componentes de dashboard]]

## Próximos passos

- [ ] Login, usuários e permissões (banco próprio já está de pé para isso).
- [ ] Módulos Folha e Patrimônio (sidebar já tem placeholders) — reusar o padrão de abas.
- [ ] Talvez repensar grupos de empresas (hoje só localStorage; poderia ser compartilhado entre máquinas).
- [ ] Possíveis análises futuras: mapa por UF, exportação Excel.

## Links

- Mapa: [[Desenvolvimento]]
