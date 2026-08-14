---
tags: [tipo/projeto, projeto/navetech-hub]
criado: 2026-07-18
status: ativo
codigo_em: ~/Dev/nexo
---

# Navetech Hub

> Plataforma web da Navecon sobre a base PostgreSQL do sistema contábil Questor (banco "Navecon" do escritório). Organizada **por módulos** (Fiscal, Contábil, e o que vier — Folha, Patrimônio): cada um com suas próprias telas e permissão. Nasceu como dashboard fiscal ("Questor BI"), virou "Questor Hub" quando "BI" ficou pequeno (é também ferramenta operacional, não só dashboard), em jul/2026 virou "Navetech Hub" e **ainda em jul/2026 virou "Nexo"** — nome/slug/banco/repositório renomeados (repo agora `git@github.com:edulanzarin/nexo.git`). Esta nota mantém o título "Navetech Hub" só pra não quebrar links; o produto é o **Nexo**.

Código em: `~/Dev/nexo` (pasta local renomeada junto com o projeto; o nome do projeto não depende dela). Remote `git@github.com:edulanzarin/nexo.git`. Slug `nexo`, containers `nexo-app`/`nexo-db`, imagem `nexo-app`, banco/role/volume/rede `nexo`, par de portas **4022 app / 5022 banco**. O sistema externo lido segue sendo o **Questor** (banco `Navecon`), intocado.

## Estado atual

Módulo **Fiscal** funcionando com dados reais, com 6 seções: Painel, Análises, Tributos, Produtividade, Conformidade, Dados. Entra-se nele pelo **launcher** (a raiz `/`), e dentro do módulo a sidebar mostra **só as seções dele** (ver "Arquitetura de módulos e permissão" abaixo).

- **Painel** (`/fiscal/painel`): resumo da movimentação — KPIs (entradas, saídas, empresas, canceladas, variação vs período anterior), resumo de devoluções/cancelamentos, evolução temporal, donut por espécie e o card de impostos (ICMS/ST/IPI/ISS/PIS/COFINS + retenções + DIFAL/FCP/FUNRURAL). Fontes dos impostos: [[Impostos no Questor - onde fica cada um]].
- **Análises** (`/fiscal/analises`): rankings/distribuições — top empresas, fornecedores/clientes, produtos, CFOPs, UF, municípios e modalidade de frete.
- **Tributos** (`/fiscal/tributos`): análise tributária das saídas — KPIs (tributos destacados, **carga tributária efetiva** = impostos÷faturamento, DIFAL+FCP a recolher, ICMS), reaproveita o `ImpostosCard` (composição ICMS/ST/IPI/ISS/PIS/COFINS + retenções, com ent/saí), **DIFAL+FCP por UF de destino** (`lctofissaidifal` × `vlricmsintufdest+vlricmsfcpufdest`), **regime tributário por CST de PIS/COFINS** (`cdsituatributpis`, tributado × monofásico × alíquota zero × isento…) e **carga por empresa** em tabela (ICMS+IPI+ST+ISS ÷ faturamento; PIS/COFINS ficam de fora do por-empresa por serem tabela separada). Endpoints `/api/fiscal/tributos-{difal,cst,carga-empresas}` + reuso de `/impostos`.
- **Produtividade** (`/fiscal/produtividade`): quanto cada **colaborador** lançou (via `codigousuario` → `usuario`). KPIs (notas lançadas, colaboradores ativos, média/pessoa, % automático — sempre focados nos humanos), **ranking em tabela de largura total** (notas ent+saí, empresas atendidas, valor movimentado, canceladas; ordenável por coluna; altura limitada com scroll + cabeçalho sticky), série notas/dia (ent×saí) e um **calendário de atividade estilo GitHub** (sempre diário, um quadrado por dia, ocupando a largura toda; cobre o período selecionado — que agora é limitado a 1 ano, ver abaixo). O usuário `0` = **Sistema** (importação automática/e-Doc) **entra no ranking** como linha própria marcada "automático" (não é colaborador, mas mostra o volume que passou pelo sistema); toggles "ocultar sistema" e "só ativos". Endpoints `/api/fiscal/produtividade{,-serie,-calendario}`.
- **Conformidade** (`/fiscal/conformidade`): saúde fiscal das saídas — KPIs de pendências (itens com **NCM inválido/genérico** `9999.99.99`/`0000.00.00` + nº de produtos a corrigir; canceladas; **denegadas/inutilizadas** via `cdsituacao`≠0; NFe/NFCe/CTe **sem chave** de 44 díg, só modelos 55/65/57), distribuição de `cdsituacao` e **ranking de empresas com mais pendências**. Descartei gaps de numeração (ruído: empresa não registra a sequência toda → 400k falsos) e itens sem CFOP (=0). Endpoints `/api/fiscal/conformidade{,-empresas}`. Consultas de item (`lctofissaiproduto`) dropam `especies` e usam `incluirCanceladas` (a tabela de item não tem `especienf`/`cancelada`); filtro de NCM inválido vai no WHERE p/ o planner reduzir por produto antes (senão `count(distinct)` sobre 1,2M itens leva ~15s).
- **Dados** (`/fiscal/dados`): todas as notas em tabela paginada, com filtros (situação todas/normais/canceladas, busca por nº/contraparte, tipo); clicar numa nota abre um **modal** com o detalhe completo (documento, modelo, chave de acesso de 44 díg. e os itens/produtos). Padrão SQL em [[Receitas SQL do Questor]].

Fiscal está bem coberto (6 seções; Recebíveis foi criada e depois removida a pedido do Eduardo).

## Módulo Contábil (iniciado)

Primeira automação de fato interligando Fiscal ↔ Contábil. Módulo tem shell/filtro próprios: **uma empresa por vez + período (≤ 1 ano)** (reusa `useFiltros`/`parseFilters`; filtro em `conf-filter-bar`).

- **Conferência Fiscal** (`/contabil/conferencia`): reconciliação — quais notas fiscais **não foram contabilizadas**. Lógica em [[Vínculo nota fiscal e lançamento contábil no Questor]] (lctoctb origem FI, `chaveorigem` = `ME`/`MS` + chave da nota). KPIs por lado (ent/saí): total, contabilizadas, **pendentes** (a lançar, com valor), "não exigem lançamento" (remessas/retornos), canceladas. Tabela **enxuta** (uma linha por nota: nº/data, contraparte, CFOP, situação, valor) com busca; clicar numa nota abre um **modal** com o detalhe — cabeçalho, resumo, divergências e os **itens/produtos**. Os itens vêm de `/api/contabil/nota-itens`, que reusa a mesma query do Fiscal por um lib compartilhado sem furar o gate por módulo — ver [[O que dois módulos compartilham é a query, não a rota]]. Endpoint principal `/api/contabil/conferencia`. Validado: empresa 1200/jun → 37 pendentes reais nas entradas (compras esquecidas), 0 nas saídas. **Duplicidade (jul/2026)**: além de pendente/conta-errada, detecta nota **contabilizada 2×** (situação "duplicada", com KPI, filtro e banner no modal) — a mesma partida em dias distintos; método e becos sem saída em [[Vínculo nota fiscal e lançamento contábil no Questor]]. Validado: empresa 264 → 18 notas re-rodadas (16/06 e 23/06), R$ 151,8k a mais. **"Contabiliza?" aprendido (jul/2026)**: se um CFOP deve contabilizar deixou de ser calibrado pelo MÊS da tela (que classificava tudo como "não exige" num mês ainda não fechado) e passou a vir de um **cadastro aprendido dos últimos 12 meses** (`conf_cfop_contabiliza`, contabiliza se lançou em ≥50% das notas), com precedência **override > aprendido > config do Questor** (a config erra dos dois lados — ver [[Plano de contabilização por CFOP no Questor]]). Semeado na 1ª conferência; botão "Atualizar do histórico" na Configuração reaprende. Validado: emp 103/junho passou de 1 pendente para 7818. **Consolidação/varejo (jul/2026)**: passou a reconhecer venda lançada em BLOCO (origem `MOV`, não nota a nota) — situação **"consolidada"** (KPI, filtro, e o detalhe aponta as contas do razão), em vez de acusar centenas de falsos pendentes; método (cruza a partida principal da nota com as contas tocadas por MOV; armadilha do override sem fórmula) em [[Vínculo nota fiscal e lançamento contábil no Questor]]. Validado emp 1015 (SANTA ORANNA)/mai: 108 falsos pendentes (R$ 207,5k) → 0.

- **Conferência de Contas** (`/contabil/contas`): o passo seguinte — entre as notas **já contabilizadas**, quais foram para a **conta contábil errada**. Compara os lançamentos FI de cada nota contra o plano do CFOP e aponta 4 tipos: conta fora do plano, lançamento faltando, valor divergente, natureza invertida (débito/crédito trocados). Endpoint `/api/contabil/divergencias`.
- **Configuração** (`/contabil/configuracao`): o plano de contabilização por CFOP, **carregado pronto do Questor** (ver [[Plano de contabilização por CFOP no Questor]]). Lista os CFOPs movimentados no período com os lançamentos esperados; editar um CFOP grava um **override** que passa a valer no lugar do plano do ERP (e a conferência cobra a versão nova); dá pra reverter pro Questor. Endpoint `/api/contabil/plano` (GET faz o merge, PUT salva, DELETE reverte). **Replicar overrides entre empresas (jul/2026)**: botão "Replicar" abre modal (base `ListaModal`) com destino + lista dos overrides (todos marcados; dá pra escolher só alguns). No destino grava como regra GERAL (estab 0) — códigos de estab não se correspondem entre empresas. Conta fixa ausente no `planoespec` do destino bloqueia o item (senão a conferência cobraria conta inexistente); destino com override no mesmo CFOP ganha "substitui existente". O POST re-deriva tudo do estado atual em vez de aceitar linhas do cliente. Endpoint `/api/contabil/plano/replicar` (GET preview, POST executa).

Duas armadilhas que quase mataram a Conferência de Contas (as duas viraram regra no código, detalhe em [[Plano de contabilização por CFOP no Questor]]): (1) **ICMS e IPI de saída são contabilizados na apuração mensal**, não nota a nota — cobrá-los por nota apontava 1170 de 1186 saídas como erradas; a solução foi só cobrar conta que comprovadamente recebe lançamento nota a nota no período. (2) **Valor só é conferível em nota de CFOP único** — com vários CFOPs não dá pra atribuir a parcela de tributo de cada um. Depois disso: 7 apontamentos em 660 entradas e 2 em 1186 saídas (1200/jun), incluindo uma nota lançada em `43763` quando o plano manda `42763` (erro de dígito) e outra em `25210` ("Gás") quando o CFOP manda `42759` ("Gás Empilhadeiras").

### Notas (jul/2026)

Seção própria na sidebar do Contábil (não é aba da Conferência) — o **mesmo explorador de notas do Fiscal** (a seção Dados), pedido pelo Eduardo pra conferir nota a nota à mão dentro do Contábil. Lista todas as notas do recorte (aqui a empresa é única, então sem coluna de empresa), com filtros de situação/contraparte/busca; clicar numa nota abre um **modal** com o detalhe completo (a linha fica enxuta, como na Conferência).

Reusa o componente `NotasTabela` **inteiro**: a única mudança foi uma prop `modulo` que troca a rota das três consultas do explorador (`/api/contabil/{notas-lista,contrapartes,nota-itens}`). As queries saíram das rotas do Fiscal para libs (`notas-lista.ts`, `contrapartes.ts`, junto do já existente `nota-itens.ts`) e cada módulo serve pela sua rota, gateada pelo caminho no `apiRoute` — o mesmo princípio de [[O que dois módulos compartilham é a query, não a rota]], que agora vale para as três consultas, não só os itens.

O detalhe da nota virou **modal** nas duas telas do explorador (era expansão inline da linha): sobra espaço pra mostrar o que não cabe na tabela — documento, modelo, empresa, chave de acesso de 44 díg. — além dos itens. `ItensNota` foi extraído pra arquivo próprio (`itens-nota.tsx`) pra os modais do explorador e da Conferência importarem sem ciclo.

**Casca de modal comum** (`components/ui/modal.tsx`): portal no `body`, ESC/backdrop pra fechar, trava de scroll do fundo e a moldura (card + cabeçalho com título e botão fechar) estavam copiados à mão em 4 modais (detalhe da conferência, explorador, contraparte, grupos). Viraram uma casca `Modal` que os 4 usam; corpo livre por `children`, largura por prop. Modal novo monta em cima dela — regra de sempre priorizar o primitivo reutilizável antes que o sistema encha de modais copiados.

**Itens robustos** (no `ItensNota`, então vale nos dois modais de uma vez): filtro de produto (código ou descrição) e linha de **somatória** — total, ICMS e IPI — que respeita o filtro (soma só o que ficou à vista). Os dois modais de detalhe ficaram mais largos (`max-w-5xl`). O de Conferência é o mesmo, só que ainda mostra as **divergências** (conta errada etc.), que são conceito só dele.

### Balancete Fiscal (jul/2026)

Seção `/contabil/balancete`: o **balancete hipotético** que as notas DEVERIAM gerar pelas regras, lado a lado com o **real** do contábil, na árvore do plano de contas — pra achar valor na conta errada em nível agregado (a Conferência acha nota a nota; isto é a lente por conta). É **movimento** do período (débito/crédito), não saldo — o saldo é consequência, ver [[Balancete é movimento do período, saldo é consequência]].

- **Motor** (`src/lib/balancete-fiscal.ts`): "replaya" cada nota pelo plano (mesmo motor da Conferência — `planoQuestor` + override + aprendido + `avaliarRegra`), somando por conta. `vlrContICMS` (token mais usado) = `sum(valorcontabilimposto)` do ICMS na `lctofisentcfop` — validado batendo com o lançamento real.
- **Honesto por construção** (a sacada que fez funcionar): só as contas onde o motor TEM regra (despesa/receita/imposto de mercadoria) mostram divergência. Todo o resto **espelha o real** (fiscal = real → sem falso positivo): contrapartida fornecedor/cliente (por pessoa, sem erro), NFSE de serviço (o fiscal define a conta na mão por nota, não tem regra — ver [[NFSE não tem regra de conta, o fiscal decide na hora]]), e as origens não-nota. O espelho é por **origem do movimento**, não por conta: nota em conta regrada = comparação; consolidação (MOV) / apuração (IM) / retenção (RE) = espelho. Assim receita com nota + cupom bate (nota pelo motor, cupom espelhado). Validado emp 1015/mai: RECEITAS 290.445 fiscal = real; só R$ 1.367 real em Custos.
- **NFSE obrigatória não lançada não some mais (jul/2026)** (`src/lib/balancete-pendentes.ts`): o espelho tinha um vão — NFSE **lançada** espelha, mas **não lançada** não tinha CFOP pro motor reproduzir NEM real pra espelhar, então um lançamento faltando ficava invisível ("tudo ok"). Como toda NFSE é obrigada, uma sem lançamento é pendência certa: prevê-se a conta pela **moda do histórico do fornecedor** e soma-se ao esperado, criando a divergência — só onde falta real, então sem falso positivo. Um **painel de pendentes** lista as notas como prova. Mercadoria/CTe não lançada já era coberta pelo motor de CFOP (reproduz independente de lançamento). Validado na 3FS (528): NFSE 7490 (NAVECON, R$672) detectada e prevista em 4538 Honorários. O detalhe do porquê (57% analítico / 88% sintética) em [[NFSE não tem regra de conta, o fiscal decide na hora]].
- **Conta errada aparece no balancete — espelho por NOTA (jul/2026)**: o espelho por conta era cego a conta errada quando a conta do plano **nunca** recebe nota (NOVARA: NFSE lançadas em 4537/4538 com plano mandando 3171, jamais usada → gate `observadas` suprimia a produção, a 4537 era espelhada, balancete "tudo ok" enquanto a Conferência achava 27 na conta errada). Princípio em [[Espelhar por balde esconde item no lugar errado]]. A correção em dois movimentos: (1) o componente PRINCIPAL (valor contábil) de nota lançada por nota **fura o gate** — a despesa/receita dela existe no contábil, então se a conta do plano nunca é usada, foi pra conta errada e o motor produz a certa; (2) quando esse bypass dispara, o lançamento real da nota (naquela natureza) **sai do espelho** — a versão do motor o substitui. Resultado: a conta do plano fica com a nota a mais (+) e a conta onde o contábil lançou fica com ela a menos (−); os pares se anulam no total (o dinheiro não some, muda de conta) e a dupla partida se mantém. Duas armadilhas no caminho: produzir sem tirar do espelho **dobra o débito** (quebrou a dupla partida em 402k na 1ª tentativa); e excluir do espelho TUDO da nota reproduzida varre componentes irmãos que o motor não reproduz (PIS/COFINS a recuperar, fase 2) → fantasma de −1,9M na 1200 — por isso a exclusão é cirúrgica, **só quando o bypass disparou**. Nota consolidada (MOV) e pendente ficam fora do bypass (`lancadas` = notas com lançamento por nota). Rotas de drill seguem a mesma regra (`observadas`/`lancadas` da empresa INTEIRA, real por natureza) — reconciliação célula ↔ modal verificada 11/11 (734/488/1200), 1015/1200 idênticas ao baseline. Os rótulos fecham a história: a conta do plano mostra "Não lançada aqui", a conta destino mostra "Conta errada" (488 conta 3095: 379 notas conta errada somando exato a célula).
- **Árvore e nível**: `classifconta` é a hierarquia (nível = nº de segmentos); botão 1..N corta a profundidade, sintética soma os filhos. **Drill-down dos dois lados** (jul/2026): clicar num valor abre os lançamentos que o compõem. O lado **real** lista os lançamentos FI (toda origem). O lado **fiscal (esperado)**, que é hipotético (não há lançamento pronto), mistura, somando EXATAMENTE a célula: **"Regra"** = as notas que o motor usou (valor esperado por nota, coletado pelo próprio motor via um `DetalheFiscal`) + **"Espelho"** = o real que o balancete espelha (MOV/IM/RE, ou nota em conta sem regra). Provado batendo a soma com a célula em 4 empresas (analítica/sintética, débito/crédito, gap 0,00). Renomeado "Fiscal (deveria)" → "Fiscal (esperado)"; "componentes fora da cobertura" → "componentes de imposto/serviço não reproduzidos" (com tooltip).
- **Armadilha do escopo do drill-down** (valia pro lado real também): escopar por `classifconta` soma irmãs erradas, porque no Questor **várias contas analíticas distintas dividem a MESMA `classifconta`** (uma por pessoa: cada cliente/fornecedor) — clicar na 142 somava as ~149 contas de cliente. Correção: linha **analítica** escopa pela própria conta; só **sintética** soma a subárvore. Detalhe em [[Vínculo nota fiscal e lançamento contábil no Questor]].
- **Diferenças** (jul/2026): não é seção nova — é a **mesma comparação**. Nasceu como aba do Balancete e depois virou um **toggle "Todas / Só diferenças" dentro do próprio Balancete** (decisão do Eduardo: pra que duas telas se o Balancete já tem a coluna Diferença? — a aba separada foi removida; ligado, o toggle mostra a lista plana das analíticas com diferença, do maior desvio pro menor). Lista as contas analíticas com diferença e, ao clicar, mostra **quais notas** a causam: por nota, o líquido (débito−crédito) esperado × o real na conta. A soma fecha com a coluna Diferença (reconciliação exata). Cada nota rotulada: "Valor diferente", "Não lançada aqui" (esperada aqui, foi p/ outra), "Conta errada" (lançada aqui, mas o plano manda outra), "Sem regra reproduzível" (sem plano — NFSE retenção ou CFOP sem tabela). Reusa o motor via um modo `net` do `DetalheFiscal`. **A distinção conta-errada × sem-plano** (jul/2026): uma nota "extra" (esperado 0 na conta aberta) TEM plano se o motor a reproduz em ALGUMA conta — mas a conta certa do plano pode não receber lançamento por nota (o gate `observadas` a suprime), então o sinal `produzidas` é coletado ANTES do gate. Ex. 488: TARCIZIO (NFE, plano manda 5922, contábil lançou 4481) = conta errada; SILVESTRE (NFSE CFOP 80010033 sem tabela no ERP) = sem plano. Ao abrir uma **sintética**, o modal mostra a coluna **Conta** (a analítica-filha onde a nota bate: a lançada quando há real, senão a esperada) — senão você não sabe em qual filha está o erro. Rodapé com **um valor só** (o total, que fecha com a célula): mostrar "total" e "a investigar" juntos confundia, porque as "sem regra" têm diferença negativa e faziam o total ficar MENOR que o a-investigar. **Faltando que foi p/ conta-sem-regra do MESMO alvo** (jul/2026): a nota esperada em 4537 e lançada na 4483 (conta sem regra → espelha, não vira diferença sozinha) parecia "não lançada aqui" e o dinheiro sumia. Correção: itemizar o real por conta em TODAS as contas alvo (o líquido da diferença segue só das regradas, pra reconciliar) — se a nota bate numa conta do alvo, é "Conta errada" apontando pra ela; só é "não lançada aqui" se o real saiu do alvo. Ex.: duas NFSE nº 7 (séries distintas — são notas diferentes) esperadas em 4537/4431, ambas na 4483. **Remanejos internos na sintética** (jul/2026): o par conta-errada DENTRO do próprio grupo (esperado = real no total → diferença 0) sumia do modal da sintética — parecia que ela escondia erro das filhas (o Eduardo clicou no 1·ATIVO esperando ver os erros da 4537, que na verdade mora no 3000). Nota com diferença ~0 mas conta esperada ≠ lançada agora vira tipo **"interno"** ("Conta errada no grupo", âmbar): listada à parte com a coluna Conta em formato "esperada → lançada" (3171 → 4537); a diferença 0 mantém a reconciliação exata, e o rodapé separa "n notas · m remanejos internos". NOVARA 3000 = 6 faltando (o que saiu do grupo) + 26 internos (o que só trocou de gaveta lá dentro). **Certa × errada explícita (jul/2026)**: o par na coluna Conta ganhou cor e rótulo — verde "certa" (a que o plano manda) → vermelho "errada" (a lançada) — e o motor ganhou o coletor `producao` (conta do componente principal por nota, PRÉ-gate), que completa o par mesmo quando a esperada fica fora do alvo aberto; a coluna aparece também em analítica quando há par. **Rodapé = a célula clicada**: em sintética grande as milhares de notas conferidas dentro da tolerância carregam resíduo de arredondamento (~R$0,06/nota, sistemático do vlrContICMS por CFOP × real) que a lista não soma — mostrar a soma das listadas divergia da célula em ~0,2%; o rodapé mostra a célula (a verdade da tabela de origem) e o tooltip decompõe (listadas + resíduo). Interno exibe "—" na coluna Diferença (o resíduo real fica no dado, só a exibição é neutra). Dá pra clicar na diferença em qualquer modo do balancete (árvore ou Só diferenças — mesmo modal, compartilhado). Os modais de nota mostram a **espécie** (NFE/NFSE); a NFSE que o motor não reproduz vem marcada **"Verificar manual"** (âmbar), deixando explícito que serviço se confere à mão. Os modais de LISTA (drill-down e culpados) partem de uma base comum **`ListaModal`** (casca `Modal` + barra de busca + tabela rolável + rodapé por slot) — cada um só monta a sua tabela e rodapé; a busca e a moldura são as mesmas (o de culpados ganhou busca de graça ao migrar).
- **Armadilha da incompletude virando falsa diferença**: onde o motor NÃO reproduz a nota (NFSE/serviço, tokens de fase 2), a conta regrada aparece com diferença que é só o motor incompleto, não erro — todas as notas caem em "extra" (esperado 0, real >0). O de serviço (emp 488 conta 4542, cursos/NFSE) é ruído. A aba detecta "toda diferença é 'extra'" e avisa que é provável incompletude, mandando conferir pela Conferência de Contas (que compara CONTA, não valor, e pega NFSE). São **complementares**: Diferenças pega valor; Conferência pega conta errada.
- **O motor SUPER-produz (o outro lado da moeda) — diferença exatamente N× é sinal**: quando o esperado é um múltiplo redondo do real (2×, 3×), quase sempre é o motor, não o contábil. Dois padrões achados (jul/2026):
  - **Nota multi-CFOP (corrigido)**: uma nota se reparte em várias linhas `lctofis*cfop`, cada CFOP com a sua parcela do valor. O motor usava o TOTAL da nota em cada CFOP → dobrava. Ex.: ELETRICA HAMILTON (488), nota R$2.276 com 2 CFOPs → esperado R$4.552. Fix: usar `valorcontabilimposto` (ICMS) de CADA CFOP; nota de 1 CFOP não muda.
  - **CFOP com apura de ST ligada, nota sem ST (corrigido)**: o CFOP 2101002 (1015) tem `apurasubtribut='1'` e a tabela de ST é IDÊNTICA à de mercadoria (as duas `D 3089 = vlrContICMS`). O motor disparava o componente de ST pelo flag do CFOP mesmo a nota não tendo ST → postava 3089 duas vezes (481,60 → 963,20). O contábil lança uma. Fix: gatear o componente `st` pela presença de ST na nota (`valorsubtribut` no PRODUTO — no `cfopTab` só tem ICMS/IPI, tipoimposto 1 e 2). 1015 passou de +1.367 a **zero** diferença.
- **NFSE PASSA a ser reproduzida — o bloqueio era `vlrContISS` compartilhado, não "atribuição" (resolvido jul/2026)**. A fórmula do débito do serviço tomado depende do CFOP: **S/ Retenção → `vlrContISS`** (valor contábil líquido de ISS); **C/ Retenção → `vlrContabil`** (esse o motor já tinha, então C/Ret sempre reproduziu — ex.: NFSE 9144/488 CFOP 8001004, plano manda D 4533, contábil lançou D 4942 = conta errada pega). O bloqueio da S/Ret: `vlrContISS` **não é exclusivo de serviço** — CFOP de mercadoria (industrialização/confecção) também tem componente com essa fórmula, então fornecer o token global disparava despesa de serviço pra TODA NFE (488 Comissões saltava a R$ 7,8 MI). **A correção certa (e a lição): ISS só existe em serviço, logo `vlrContISS` só deve ser fornecido quando `especie = 'NFSE'`.** Com o gate por espécie, a NFSE reproduz e compara (488 Comissões 4062 fiscal=real, dif zero), pega anomalia real (HODECKER 4542→4950), sem super-produção; e as "Verificar manual" caem só pras NFSE que ainda usam token de fase 2 (retenção pura). A tese antiga ("o motor não sabe atribuir serviço à conta") estava **errada** — a atribuição vem do CFOP no plano, que é confiável; o problema era só o token vazando pra mercadoria.
- **Culpados reconciliam em sintética (fix jul/2026)**: clicar na diferença de uma sintética somava demais (390k vs 57k) porque a subárvore mistura conta com regra (gera diferença) e sem regra (espelha → diferença 0); o real das sem-regra entrava como falsa diferença. Correção: restringir a comparação às contas que o motor de fato regra (`DetalheFiscal.regradas`). Agora clicar em qualquer diferença (sintética ou analítica) soma exatamente a célula.
- Escopo restante: a completude 100% do motor (imposto/retenção nota a nota) segue grind aberto — a decisão é que o valor real é **detecção de anomalia**, não reprodução perfeita. Contas com muitas notas (>2000 linhas) truncam a lista do drill (soma parcial, sinalizada).

### Balancete e Análise (jul/2026)

Seção **"Balancete Contábil"** com **duas abas do mesmo trabalho** (mesmo filtro empresa+mês): **Balancete** (botão "Gerar") monta a **balancete de verificação contábil** de verdade — por conta do plano: conta reduzida + natureza (D/C), descrição/classificação, saldo anterior · débito · crédito do período · saldo atual, na árvore do `classifconta` com corte por nível, igual o Questor imprime (`resumirBalanceteVerificacao` sobre o mesmo `coletarBalanceteContabil`, que fecha ao centavo); e **Análise** (botão "Analisar") roda o motor/laudo sobre ela. Mostra só contas com saldo ou movimento (as zeradas já saem no coletor); sem linha de totais — consolidar é papel da Análise. O `id` da seção segue "analise" (chave de permissão), só o rótulo/caminho mudaram.

Duas armadilhas de navegação resolvidas nessa entrega, que valem pros próximos módulos com abas: (1) **aba de uma seção precisa morar sob o prefixo do path da seção** — a sidebar casa "active" por `pathname.startsWith(secao.path + "/")`, então a Análise foi de `/contabil/analise` pra `/contabil/balancete-contabil/analise` (e o Balancete Fiscal saiu do genérico `/contabil/balancete` pra `/contabil/balancete-fiscal`); senão o item some do active ao trocar de aba. (2) **o marcador de executado (`ap`) é por ABA, não por seção**: ao navegar entre abas a shell leva os filtros mas remove o `ap`, então cada aba só consulta no clique do seu botão — estar no mesmo menu não a dispara (o dado segue em cache, re-clicar é instantâneo). Refina o padrão execução-por-botão.

### Análise de Balancete (jul/2026)

Seção `/contabil/analise`: gera um **laudo do balancete** (pontos fortes, fracos, alertas, recomendações) pra apresentar ao cliente da contabilidade — a primeira feature do Hub com **IA generativa** (Claude). Segue [[Coleta determinística, LLM só interpreta]]: o backend monta o balancete de saldos **sem IA** (`balancete-contabil.ts`: saldo inicial + movimento e saldo por mês, a partir de `lctoctb` + `planoespec`, por empresa) e manda os dados organizados ao Claude, que escreve o laudo. Modelo padrão **Sonnet 5** (`claude-opus-4-8` no modo premium via `ANALISE_MODELO`), **saída estruturada** em JSON schema (cai direto na UI/PDF) e **system prompt cacheado**. O backend só seleciona/compacta (todas as sintéticas + as 80 maiores analíticas) e deixa a IA calcular os indicadores — grupos de conta variam de plano pra plano, então ler os rótulos é mais robusto que cravar "classe 1 = ativo". Custo ~R$1–4 por análise (12 meses).

Decisões: usei `lctoctb` (colunas documentadas em [[Módulo contábil do Questor]]) em vez de `saldoctbmensal` porque não tinha acesso ao banco pra validar o schema mensal — duas premissas contábeis (`tipolancamento='LN'`, saldo inicial acumulado) ficaram comentadas no código pra validar contra um balancete conhecido. Período por **mês** (balancete é mensal): `PeriodoMensalDropdown`, com presets ancorados no **último mês fechado** (não se analisa mês corrente incompleto — pareceria queda). UI mostra o laudo na tela e exporta **PDF via impressão** (CSS `@media print` isola o laudo). Sidebar do Contábil reordenada nessa entrega: Conciliação virou a 1ª e a home do módulo. Pendente: `ANTHROPIC_API_KEY` (o Eduardo ia pedir ao responsável) — sem ela a rota devolve erro claro.

**Motor determinístico + correção contábil (jul/2026):** depois a análise deixou de pedir indicadores à IA e passou a um **motor determinístico** (`analise-motor.ts`, custo zero, roda no Executar) que classifica, calcula e valida; a IA só redige a prosa do laudo sobre o que o motor achou (`analise-balancete.ts`, botão sob demanda). Com acesso ao banco, a classe-por-número deixou de ser frágil: o plano-padrão foi **validado em ~1310 empresas** e documentado em [[Plano de contas padrão do Questor e leitura do balancete]] — isso reverteu a decisão anterior de "deixar a IA ler os rótulos", porque determinístico ficou ao mesmo tempo robusto, de graça e auditável. Três bugs contábeis reais corrigidos, confirmados contra empresas reais (o balanço fecha à casa do centavo): (1) **o balanço não fechava na tela** — mostrava o PL registrado (2.4/2.5/2.6), que num balancete pré-apuração não inclui o resultado do exercício, então Ativo ≠ Passivo+PL; agora o PL é o **residual** (Ativo − Passivo exigível), embute o resultado e o balanço fecha, com o não-transportado virando alerta; (2) **evolução mês a mês enganosa** — contas de resultado acumulam o ano no saldo, então virava reta sempre subindo; receita/despesa/resultado passaram a usar o **movimento do mês** e ganharam uma **DRE em cascata** do período; (3) **alertas afogados em ruído** — o "sinal atípico" listava cada cliente/fornecedor (adiantamento = normal), agora colapsado num resumo (contra-parte = muitas irmãs sob o mesmo classif), sobrando só o impossível (caixa/estoque negativo) e o estrutural. Princípio por trás do (2): [[Balancete é movimento do período, saldo é consequência]]. Indicadores novos: liquidez seca/imediata (estoque/disponível reais) e retorno sobre o PL.

**Laudo trocou pro Groq + sigilo do cliente (ago/2026):** o laudo escrito deixou a API paga da Anthropic e passou pra **Groq** (tier grátis, endpoint OpenAI-compatível, `llama-3.3-70b-versatile` por padrão), chamada por `fetch` — decisão de não pagar Anthropic por ora; o motor determinístico segue intocado e de graça. Antes do envio, **nome e CNPJ da empresa são censurados**: o payload monta com `[EMPRESA]` na origem, mais uma varredura que mascara nome/CNPJ/CPF que escape, sem corromper valores em R$; o nome real é reposto só no laudo que volta, então a IA nunca vê a identificação — sigilo do cliente e LGPD. Técnica em [[Censurar a identificação antes de mandar pro LLM externo]]. Envs `GROQ_API_KEY` (no `.env` gitignored) e `ANALISE_MODELO`; o "Pendente: ANTHROPIC_API_KEY" acima fica superado. Na mesma leva, tunables de sessão e do pool do banco saíram do código pro `.env` (`SESSAO_DIAS`, `DB_POOL_MAX/IDLE_MS/CONN_MS`, `QUESTOR_/APP_DB_STATEMENT_TIMEOUT_MS`), mantendo os defaults atuais — [[Configuração vem do ambiente, não do código]].

### Conciliação

Segunda automação do Contábil: ler extrato bancário (OFX/PDF) e gerar os lançamentos já na conta certa. Leitura, prévia **e exportação funcionando** (ago/2026).

Duas abas, `Importar` (a raiz, é o dia a dia) e `Regras` (cadastro).

- **Importar** (`/contabil/conciliacao`): envia OFX ou PDF, aplica as regras e mostra a prévia dos lançamentos, com KPIs e filtro prontos/pendentes. Lê **OFX de qualquer banco** e **PDF de 9 bancos** — ver [[Ler extrato bancário em PDF]]. Numa pendência dá para escolher a conta ali mesmo (vale só para aquela importação). **Salvar como regra (ago/2026)**: ao escolher a conta à mão, um botão cadastra aquela descrição (sem o sufixo de data volátil) como regra parcial da conta, no sentido certo — a próxima importação casa sozinha (com Reaplicar regras vale já na prévia atual). É o mesmo espírito de [[De-para determinístico com override que vira aprendizado]]: a correção manual vira aprendizado persistente. O extrato carregado sobrevive à navegação dentro do módulo (troca de aba **e de seção**) e só é liberado ao sair do módulo (jul/2026 — antes caía ao trocar de seção; ver [[Estado de tela pertence à seção, não à página]]).
- **Exportar (ago/2026)**: gera o arquivo `.nli` de importação do Questor a partir dos lançamentos prontos (só os resolvidos; a pendência fica de fora). Diferente da Implantação, cada lançamento já é dupla partida completa (banco de um lado, contrapartida do outro), então vira uma linha direta com a SUA data — sem transitória. Filial e histórico na própria tela, download no navegador, geração auditada, contas validadas contra o plano antes de gerar. Formato em [[Para alimentar o ERP, gere o arquivo de importação dele]] — o `.nli` saiu do gerador da Implantação para `src/lib/nli.ts` e as duas telas reusam.
- **Cadastro** (`/contabil/conciliacao/regras`): por conta de banco (escolhida no plano do Questor, restrito a 1.1.01 — ver [[Contas bancárias e layout de contabilização no Questor]]), uma lista de regras `termo → contrapartida`, separando **pagamento** de **recebimento** (pode ter só um dos dois). Casamento **exato** ou **contém**.
- **Casamento** (`src/lib/regras-extrato.ts`): normaliza acento/caixa/espaço, então "MAGALHÃES" casa com o termo "MAGALHAES". Quando várias regras casam, vence a **mais específica** — exato ganha de parcial, e entre parciais o termo mais longo ganha. Isso evita ter que gerenciar ordem de prioridade na mão.
- **Replicação**: copia as regras de uma conta de banco para outras contas da mesma empresa ou para outras empresas, avisando quais contrapartidas não existem no plano do destino (o plano é por empresa).
- As contas são **validadas contra `planoespec`** ao gravar — dígito errado é recusado na hora, em vez de estourar só na importação.

O **arquivo de saída** usa o mesmo `.nli` ("Layout Importação Lançamento Contábeis") já provado na Implantação — não era preciso um formato novo.

Ressalva registrada: o extrato do **Bradesco** (conta escrow) lê todas as linhas com os sinais certos, mas o "SALDO" do rodapé não reconcilia com saldo inicial + lançamentos — parece ser posição de títulos, não caixa. Confirmar com o setor antes de usar em produção.

**Bradesco "Extrato Mensal / Por Período" (ago/2026)**: o layout do Net Empresa não cabia no motor tabular — a data só aparece na 1ª linha do dia e o lançamento ocupa 2–3 linhas — então lia **um lançamento por dia**, e ainda casava por engano no config genérico por causa de "BRADESCO SEGUROS" numa descrição. Leitor de layout próprio `bradescoMensal` (âncora na linha de valor, valor pela diferença de saldo, tem precedência sobre o config genérico), validado contra um extrato real: **195 lançamentos, créditos/débitos batendo exatamente com os totais impressos**. Método em [[Ler extrato bancário em PDF]].

### Implantação de Saldos (jul/2026)

Seção `/contabil/implantacao`: quando uma empresa entra no escritório, lança o **balancete de abertura** da contabilidade anterior no plano de contas dela no Questor. O Nexo **não escreve no Questor** (produção, read-only) — prepara o **arquivo de importação** e o contador importa lá. O layout é o mesmo `.nli` que faltava na Conciliação (`knowledge/layout importacao lancamentos contabeis`): uma partida por linha, `C;empresa;estab;data;contaDeb;contaCred;histórico;complemento;valor`, delimitado por `;`, decimal com vírgula, `TIPOLANCAMENTO='LN'`/`ORIGEMDADO='3'` fixos pelo layout. Princípio em [[Para alimentar o ERP, gere o arquivo de importação dele]] — a Conciliação pode reusar o mesmo gerador.

O problema real (dito pelo Eduardo): entram muitas empresas por mês, de softwares diferentes, e a numeração das contas nunca bate — **e eles sempre mandam PDF, nunca planilha**. A saída para isso: **a saída é uma só, a entrada é que varia**. Um **formato canônico** (`LinhaOrigem`) no meio — o parser multiplica na frente, mas de-para, validação e gerador são escritos uma vez. A entrada é **PDF** via `pdftotext -layout` (a mesma lib da Conciliação, extraída pra `src/lib/pdf-texto.ts`).

- **Parser de PDF** (`src/lib/implantacao-pdf.ts`): em vez de um leitor por software (como o extrato faz por banco), usa a **posição das colunas** que todo balancete compartilha — código à esquerda, **saldo atual é o último valor da linha** (a coluna mais à direita nos dois layouts vistos). Tolera duas manhas do `pdftotext`: a 1ª célula vindo com "código classificação" juntos (só 1 espaço entre eles) e o código reduzido se perdendo, deixando a classificação pontuada como chave. Os helpers de saldo (D/C por sufixo/parênteses/sinal) e o **filtro de sintéticas por hierarquia de prefixo** (só analíticas entram, senão os grupos duplicam) ficam em `implantacao-entrada.ts`, valendo pra qualquer origem. Natureza vem da origem quando marcada, senão do plano de destino (Systemar não marca D/C, Patrimonium marca).
- **O limite honesto do PDF genérico**: balancete **por empresa** e limpo (Systemar/Questor) sai redondo; balancete **consolidado** (o Patrimonium de teste, 5 empresas) o `pdftotext` mangle — joga valor em linha sem código e código em linha sem valor, e a extração vem incompleta. Não é heurística que conserta: é a tabela destruída na extração. A **rede de segurança é o fechamento** (ver abaixo) — não some silenciosamente.
- **De-para** (`src/lib/implantacao-depara.ts`): cascata determinística (sem IA) override salvo → classificação → descrição normalizada (Dice, restrita a mesma classe e natureza). Detalhe reutilizável em [[De-para determinístico com override que vira aprendizado]].
- **Gerador** (`src/lib/implantacao-gerar.ts`): uma linha por conta contra uma **conta transitória de implantação**; débito/crédito pelo lado da natureza; a transitória zera quando o balancete fecha (`Σdeb = Σcred`). **O fechamento é o guardrail contra PDF mal lido**: a tela mostra um indicador PERSISTENTE (débitos × créditos) e, se não bate, avisa que a leitura veio incompleta ANTES de gerar. Testado ponta a ponta com os dois balancetes de exemplo.
- **Escolhas na tela**: **data dos lançamentos** (a data de entrada da empresa), **conta transitória** e **código de histórico** — todos selecionáveis na barra de parâmetros. O **histórico já vem com padrão fixo `350`** ("Valor Referente [DICTEXTO]", o de implantação usado na prática); como tem marcador `[DIC]`, **pede complemento** e o campo já abre com "IMPLANTACAO DE SALDOS". (Histórico sem `[DIC]` some o campo de complemento.) Nota: existiu uma fase com **padrão por empresa** em `implantacao_config` (o medo de fixar em código); em jul/2026 o Eduardo optou por **fixar o 350 na tela** — a tabela `implantacao_config`/rota ainda existe mas a tela não a carrega mais. O de-para confirmado à mão vira override em `implantacao_depara` (banco do app; nada disso toca o Questor). Geração é evento auditável (`contabil.implantacao.gerar`). A seção é o **último** item da sidebar do Contábil (trabalho pontual de entrada de empresa, não do dia a dia).
- **A prévia (tabela) só mostra o que varia por linha**: Origem, Débito, Crédito, Valor, Situação. Data/Histórico/Complemento saíram — eram constantes (vêm da barra), só poluíam. Os 4 KPIs (Contas/Casadas/Confira/Sem conta) são **botões que filtram** a tabela pela situação (clicar de novo desliga) — atalho pra caçar as sem conta antes de gerar. O filtro preserva o índice original das linhas, então editar/escolher conta continua batendo certo.
- **Por que lançamentos e não "saldo direto"**: o Questor tem Implantação de Saldos nativa (tela "Cadastro: Saldos Contábeis" → `implsaldoctb`, saldo por conta sem contrapartida/histórico) — ver [[Módulo contábil do Questor]]. **Mas essa tela não tem importador** (verificado jul/2026, digitação manual conta a conta). Por isso o Nexo importa **lançamentos** (a única tela com layout de importação), aceitando a transitória + histórico como custo do caminho automatizável. Descartada a ideia de gerar o formato nativo — não há como importá-lo.
- **Pendente**: leitores de PDF **dedicados por software** para os layouts que o parser genérico não pega (consolidado etc.) — o padrão do extrato (um leitor por banco) aplicado ao balancete; validar o arquivo gerado importando de fato num Questor de teste (o formato do valor — com/sem separador de milhar — a confirmar contra uma importação real).

### Fechamento, Contas de Controle e Provisões (jul/2026)

Três automações pedidas pelo Eduardo depois de o Contábil já ter as conferências e os balancetes — a ideia dele era "que mais dá pra automatizar". Todas na branch `feat/contabil-fechamento`.

- **Fechamento mensal** (`/contabil/fechamento`): o **semáforo "posso fechar o mês?"**. Não é motor novo — **orquestra os motores já validados** (o determinístico do balancete + a conferência fiscal de entradas e saídas) num veredito verde/amarelo/vermelho por empresa+mês, e cada checagem linka a tela onde a pendência se resolve (com `&ap=1` pra cair já executada). Regra: balancete que não fecha ou conferência com nota não-contabilizada/conta-errada trava (vermelho); duplicidade é amarelo. Custo zero. Princípio em [[Fluxo de fechamento é orquestração dos motores que já existem]]. Pra isso, o núcleo `conferir()` (580 linhas) saiu da rota `/api/contabil/conferencia` pra uma lib `conferencia-servico.ts` que a rota e o fechamento compartilham — extração-na-hora, mesma doutrina de [[O que dois módulos compartilham é a query, não a rota]].
- **Contas de Controle** (`/contabil/contas-controle`): **composição do saldo por origem** (escolha do Eduardo entre 3 escopos possíveis). Abre o movimento do mês de cada conta patrimonial (classe 1/2, fora de compensação e PL) por `codigooriglctoctb`, agrupado em baldes de leitura (**fiscal FI, folha FP, financeiro FN/CP/CR, patrimônio IM, manual CB/AA/XX…**). O valor de conciliação: uma conta de controle devia ser alimentada e baixada por MÓDULO, então **lançamento manual numa conta que deveria fechar sozinha acende o alerta** (KPI "com ajuste manual" + badge + filtro "só manual"). Robusto porque **não chuta a classificação da conta** (evita o "conciliado silenciosamente falso" que [[Plano de contas padrão do Questor e leitura do balancete]] alerta) — mostra o movimento e deixa o manual saltar. Reusa `coletarBalanceteContabil` (saldo/classif/natureza); só a agregação por origem é nova.
- **Provisões** (`/contabil/provisoes`): **folha calculada × contábil lançado** (escolha "conferir", não gerar arquivo — só leitura). O Questor calcula a provisão de férias/13º como **folha própria** (`tipocalculo` 70/71, resultado em `calculoevento`) e ao integrar vira lançamento origem `FP` nas contas de provisão — ver [[Módulo de folha e eSocial do Questor]]. A tela confronta o **calculado** (Σ dos proventos das folhas 70/71 que fecham no mês) com o **lançado** (movimento credor FP em contas cujo nome contém "provis") e mostra a divergência. A identificação (quais tipos de folha, quais contas) é **heurística a validar no banco real** — por isso a tela **deixa o método visível** (lista as folhas e as contas que entraram na conta) em vez de só cuspir um número; padrão em [[Deixar o método da conferência visível quando o SQL não foi validado]].

Pendências dessas três (Eduardo valida no banco read-only, que eu não tinha aqui): as queries novas do #3 (agregação por origem) e do #4 (folhas 70/71 e contas "provis") não foram rodadas contra o Questor — compilam (tsc+eslint+build), mas os números precisam bater contra uma empresa conhecida. O #4 em especial: confirmar se somar os proventos (`tipoevento=1`) das folhas 70/71 é a base certa da provisão, e se casar conta por nome "provis" pega todas.

### Auditoria de Lançamentos (jul/2026)

Seção `/contabil/auditoria` (branch `feat/contabil-auditoria`): varre o `lctoctb` do período (uma empresa por vez) e agrupa os lançamentos com anomalia por tipo de achado — o nível que o balancete agregado **esconde** ([[Auditar o registro, não só o agregado]]): um lançamento em conta sintética some do rollup (o coletor do balancete só redistribui analíticas), uma conta órfã não aparece em conta nenhuma. Complementa o `analise-motor` (que audita **saldos** por conta), não o repete — a lente é a partida individual.

Seis checks determinísticos sobre a linha (deb/cred na MESMA linha no Questor, então "partida não fecha" não existe por construção — o que se caça é a perna caída em conta errada): **conta sintética** recebendo lançamento, **conta fora do plano** (órfã), **sem histórico** (código nem complemento — a ECD I200 exige), **ajuste de período anterior** (origem `XX`/`AA`), **manual em conta de controle** (`CB`/`IP`/`LA`/`ZZ` numa conta patrimonial — reusa o `ehControle` das Contas de Controle) e **partida repetida** (mesma data/contas/valor/histórico → dupla contabilização). Cada grupo traz contagem e valor **exatos** via `count()/sum() over()` + `LIMIT` numa query só (total real antes do corte da amostra) e uma **amostra** navegável — a memória de cálculo que deixa o método auditável na própria tela, porque duplicidade e "manual em controle" são heurísticas a validar ([[Deixar o método da conferência visível quando o SQL não foi validado]]). Seção própria no fim da sidebar; admin vê por padrão (`podeSecao`), o gate de rota entrou no `api-secoes`. Schema em [[Módulo contábil do Questor]]; nomes de coluna **conferidos contra o código existente** do Nexo — o que pegou um bug antes de rodar: `historicoctb` e `usuario` são GLOBAIS no Questor, não por empresa (a query filtrava por `codigoempresa` e quebraria).

Compila (tsc+eslint+build). **Pendente**: validar os números contra uma empresa conhecida no banco read-only (as heurísticas de duplicidade e manual-em-controle, e as colunas `chavelctoctb`/`datahoralctoctb` que a doc do Brain traz mas o código ainda não usava). **Fase 2** possível (fora do MVP por decisão de escopo): defasagem digitação × competência, valor redondo suspeito, lançamento em período já fechado, e um **laudo IA** priorizando/explicando os achados ([[Coleta determinística, LLM só interpreta]]).

### Central de Pendências (jul/2026)

Seção `/contabil/pendencias` (branch `feat/contabil-pendencias`): uma **fila única de exceções** que junta as pendências da **Conferência** (nota não contabilizada, conta errada, duplicada) e os achados da **Auditoria de Lançamentos** (sintética, órfã, sem histórico, extemporâneo, manual em controle, partida repetida), com **triagem** (resolver/ignorar/reabrir) que persiste. Nasceu de analisar o **"Analista Contábil Digital"** da Questor — um robô que LÊ documento (OCR) e LANÇA no ERP: como o Nexo é read-only, a tradução honesta não é copiar o motor de escrita, é ser a **auditoria/fila de exceção** dessa automação (quem adota o robô ganha, no Nexo, quem pega o erro dele).

Orquestra os motores já validados (como o Fechamento — [[Fluxo de fechamento é orquestração dos motores que já existem]]): reusa `montarAuditoria` e extraiu `conferirTudo` (a conferência COMPLETA, sem paginação de tela) de `conferir`, que segue com a mesma assinatura pra tela de Conferência e o Fechamento. O novo é a **triagem**: os achados são recomputados ao vivo do Questor a cada carga (nada de achado gravado); só a decisão humana grava, em `conf_triagem` (migration 017), com **identidade estável** `(fonte, chave, tipo)` — `ME`/`MS`+chave da nota na conferência, `chavelctoctb` na auditoria — e trilha em `auditoria`. Padrão em [[Fila de exceção recomputa o achado e persiste só a triagem]]. Uma empresa por vez (reusa as varreduras das telas de origem, mesmo custo). É outra lente da doutrina [[Auditar o registro, não só o agregado]], agora virada pra ação (não só ver o achado, tratá-lo).

Verificado: tsc/eslint limpos, migration aplicada e tabela conferida, smoke HTTP (401 sem sessão, 307→login na página, 405 no GET da triagem POST-only) e o upsert+select da triagem exercitado direto na tabela. **Pendente** (precisa de sessão real, que não dá pra dirigir daqui): o conteúdo do payload numa empresa conhecida — mas a leitura do Questor reusa `conferir`/`montarAuditoria`, já validados nas telas de origem.

Primeira de 3 fases de uma análise do Analista Contábil Digital (as outras, planejadas: **conta provável pela contraparte** no detalhe da nota divergente; **cobertura de contabilização por origem** — folha/financeiro que entrou e não virou lançamento, guiada pela dessincronia da integração `FP` da [[Módulo de folha e eSocial do Questor]]).

Próximos passos possíveis no Contábil: balancete/DRE/razão (usar `saldoctb`/`lctoctb`/`planoespec`), mais conferências. Ver [[Módulo contábil do Questor]] e o mapa [[Banco Questor]].

Devoluções e cancelamentos hoje entram como **resumo** no Painel (os endpoints detalhados existem no código, se um dia virar seção própria de novo). Apuração foi tirada: era estimativa gerencial (débito−crédito), não a oficial do SPED — ver nota em [[Impostos no Questor - onde fica cada um]].

Filtros compartilhados no topo (período, empresas multi-seleção, espécie, **grupos** criados no app via localStorage) — na URL e **preservados ao navegar entre seções**. **Persistência por módulo (jul/2026)**: o recorte (filtro/`ap`) e o estado de tela (busca, extrato/prévia carregados) agora vivem enquanto se navega pelo módulo e caem só ao sair dele, alinhando Contábil/Fiscal/Folha ao RH — antes o estado de tela caía a cada troca de seção, obrigando a recarregar. Detalhe e porquê em [[Estado de tela pertence à seção, não à página]]. O toggle **Valor | Quantidade** só aparece onde muda algo (Painel e Análises). Tema claro/escuro.

**Execução por botão (jul/2026)**: mudar filtro não dispara consulta — o usuário edita um rascunho e o botão da barra comita para a URL (`ap=1`), e só então as queries rodam. O verbo do botão vem do catálogo de abas: "Executar" (Conferência, telas fiscais), "Carregar" (Configuração do plano). Na **Conciliação** os controles (conta, extrato, senha, Replicar) moram **na linha da barra**, ao lado da empresa — renderizados pelo shell via slot `extras` da ConfFilterBar, compartilhando o estado da seção com a página (o store ganhou ouvintes + `useSyncExternalStore`, ver [[Estado de tela pertence à seção, não à página]]). Escolher o extrato só guarda o arquivo; o **Executar da barra** é que processa (escolher ≠ executar). O botão vira spinner enquanto roda. Padrão em [[Consulta pesada executa por botão, não por mudança de filtro]].

**Período limitado a 1 ano** (`MAX_DIAS_PERIODO` em `fiscal-filters.ts`): evita consultas pesadas nas tabelas gigantes e trava. Guardado em duas camadas — `parseFilters` recusa (`400`) qualquer range > 366 dias em todos os endpoints, e o seletor de período personalizado no `filter-bar` já limita o fim a 1 ano (toast avisando). Os presets todos cabem em 1 ano.

**Padrão de navegação (reusar nos próximos módulos)**: **launcher** na raiz `/` escolhe o módulo (só entram na grade os que a sessão libera); dentro do módulo, a sidebar é **escopada** — só as seções dele, com "Trocar módulo" pra voltar ao launcher. O `layout` de cada módulo monta essa sidebar e segura header + filtros compartilhados; cada seção é uma página que busca só os seus dados. Catálogo dos módulos (fonte única do launcher, sidebar e gate) em `src/lib/modulos.ts`; seções em `src/lib/fiscal-secoes.ts` / `contabil-secoes.ts`. Isso substituiu a sidebar única em acordeão sobre todos os módulos — o porquê da troca em [[Sidebar em acordeão e layout de módulo]].

### Arquitetura de módulos e permissão

O **seam de permissão** foi cravado antes do login (`src/lib/sessao.ts`) e em **jul/2026 o login real entrou** — exatamente como previsto, **só o stub de `getSessao()` mudou**, nada foi retrofitado (launcher, layouts e `apiRoute` já passavam por ele). Hoje `getSessao` lê o cookie opaco → linha `sessao` no banco do app → `usuario` + perfil. Padrão em [[Cravar o seam de permissão antes do login]] e [[Sessão opaca no banco separa autenticação de permissão]]; doutrina em [[Permissão se valida no servidor, não na interface]].

**Três eixos de autorização** (migration `004_auth.sql`):
1. **Módulo** — derivado (ter ≥1 seção nele).
2. **Seção** (`view`/`edit`) — o grão: cada aba da sidebar tem nível próprio, em `usuario_secao`. O `apiRoute` autoriza por seção via um **registro único endpoint→seção** (`src/lib/api-secoes.ts`), porque as rotas são namespaceadas só por módulo e algumas são compartilhadas (ex.: `/api/fiscal/impostos` serve Painel e Tributos). Admin tem acesso total.
3. **Empresa** — o usuário só vê empresas de **grupos reutilizáveis** (`empresa_grupo`) + extras individuais, ou `todas_empresas`. Aplicado no servidor num funil só: `buildWhere` (fiscal/contábil) e `construirBase` (folha, que filtra por estabelecimento) viram session-aware e clampam — nunca confia na lista do cliente. Ver [[Escopo de dado se clampa no servidor, num funil só]].

**Ponto cego do escopo fechado (jul/2026)** — auditoria pela pergunta "nenhuma função acessa empresa fora do escopo?". Um monte de rota de **empresa avulsa** do Contábil (`plano`, `conferencia`, balancetes, `bf-check`, `aprender`, `contas`, `extrato-*`) e o `nota-itens` (Fiscal/Contábil) liam `?empresa=` cru e consultavam o Questor sem re-checar o escopo — só tinham o gate de módulo, então quem tinha o módulo lia dado de qualquer empresa trocando o parâmetro. Novo `assertEmpresaVisivel` (em `api-route.ts`) barra empresa fora do alcance da sessão (o escopo do cargo), aplicado em toda rota de empresa única. Lição em [[Drill-down por id foge do funil de escopo e precisa de gate próprio]].

Uma volta atrás no caminho: cheguei a esconder as empresas do **RH** (NAVECON/FOUR/FINAVE) de todo mundo fora do RH — errado. A regra é que elas **seguem o cargo** como qualquer empresa (quem tem no cargo vê nos outros módulos também); o que é fixo é o próprio **módulo RH**, que sempre mostra as 3 via escopo forçado (`empresasDoFiltro`/`EMPRESAS_RH`). O guard acima, por usar o escopo normal do cargo, já faz isso certo sem caso especial.

Login por email/senha (Server Action, `scrypt` do `node:crypto`), `proxy.ts` (ex-`middleware`, redirect otimista pro `/login`), e **área `/admin`** (fora do catálogo de módulos, só admin) pra gerir usuários, perfis por seção e grupos de empresa. Validado ponta a ponta: 401 sem sessão, 403 em seção sem acesso, lista de empresas recortada e empresa forjada retorna vazio.

**Permissão 100% do cargo, multi-cargo (jul/2026, migration 010)**: o modelo dos "três eixos" acima ganhou ajuste fino por usuário (override de seção em `usuario_secao`, grupos/empresas avulsas por pessoa) e virou exceção por gente. Reformulado: **toda a permissão vem do CARGO** (seções + grupos de empresa + os flags `admin` e `todas_empresas`, que saíram do usuário e viraram colunas do cargo), e a pessoa passa a ter **vários cargos** (`usuario_cargo` N:N) — o acesso é a **UNIÃO** deles. Sem ajuste por usuário: precisou de algo diferente, cria/atribui outro cargo. "Admin" agora é um **cargo** de acesso total (criado idempotente no seed e vinculado ao usuário admin). A sessão deriva admin/seções/empresas com `where cargo_id = any($cargos)`. Form de usuário virou só o seletor de cargos (múltiplo); form de cargo ganhou os dois checks. As tabelas de override (`usuario_secao`/`usuario_grupo`/`usuario_empresa`) e as colunas antigas do usuário ficaram mortas (dropar depois). Padrão em [[Permissão composta por papéis somados, não exceção por usuário]].

**Perfil de usuário** (migration 005, feito pra crescer): além de nome/email, tem cargo, setor (rótulo, não regra), telefone, último acesso e **foto de perfil** — a foto mora numa tabela `usuario_avatar` (bytea, self-contained, sem storage externo) e é servida por rota com checagem de sessão, aplicando [[Servir anexo por rota com checagem de permissão]]; o `Avatar` cai pra iniciais quando não há foto. Grupo de empresas padrão **"Todas menos NAVECON"** (1476 empresas) via `scripts/seed-grupo-padrao.mjs` — snapshot, reconcilia rodando de novo.

**Estado de tela por seção** (`src/lib/estado-secao.ts` + o hook `useEstadoSecao`): busca, filtros, paginação e o extrato já carregado **sobrevivem à navegação dentro da seção** (trocar de aba mantém) e são descartados ao sair dela — quem descarta é o shell do módulo, no cleanup do efeito, porque é ele que sabe qual seção está ativa. Vale nos dois módulos. O padrão e as armadilhas em [[Estado de tela pertence à seção, não à página]]; o motivo de não usar o cache do React Query para isso (generalizou o que a Conciliação fazia à mão) em [[Cache do React Query não é lugar de estado de interface]].

O conhecimento do banco Questor (schema, impostos, canceladas, devoluções, SQL) vive aqui no Brain em `03 - Recursos/Banco Questor` — consultar de lá para novas automações.

## Módulo Folha (iniciado)

Terceiro módulo, primeiro sobre o **RH/folha** do Questor (tabelas `func*` — ver [[Módulo de folha e eSocial do Questor]]). Reusa a casca de módulo inteira: entrou como uma linha no catálogo `modulos.ts` (`ativo: true`, home `/folha/rotatividade`), o gate de sessão liberou `folha`, o `apiRoute` passou a casar `/api/folha/...`, e launcher + sidebar (dirigidos por `MODULOS`) propagaram sozinhos. Shell próprio como o Contábil: **uma empresa por vez + período (≤ 1 ano)** via `ConfFilterBar`/`useFiltros`, execução por botão.

- **Rotatividade** (`/folha/rotatividade`): o **turnover** por empresa, pedido pelo DP. Índice ((admissões + desligamentos) / 2 ÷ colaboradores ativos × 100) a partir de `funccontrato` — admissão/desligamento por `dataadm`/`datadem`, ativos numa data = admitido até ela e ainda não desligado. **Denominador alinhado ao relatório do DP**: ativos = efetivo no FIM do intervalo (não a média de livro), validado batendo número a número com o print de referência (setor 7 ativos, 1 adm, 1 dem = 14,29%). Várias lentes: **KPIs** (turnover %, admissões, desligamentos, colaboradores ativos), **série mensal** (gráfico composto: barras adm/dem em `--good`/`--critical` + linha do índice em `--esp-5`), **motivo do desligamento** e **tempo de casa dos desligados** (barras), e duas **quebras** — por **organograma (setor)** e por **cargo** — em tabela com efetivo/adm/dem/turnover, filtros por faixa (Todos/Com movimentação/Alto>10%/Médio 2–10%/Baixo<2%), busca, ordenação, badges por faixa e linha de total que fecha com os KPIs. Fontes: setor por `funclocal.classiforgan` (vigência `datatransf`) → `organograma.descrorgan`; cargo por `funccargo` → `cargo.descrcargo`; motivo por `rescisao.codigocausa` → `causademissao.descrcausa`. A tabela é um componente genérico (`RotatividadeQuebra`) que setor e cargo reusam; motivo e tempo de casa reusam um componente de barras (`RotatividadeBarras`). Método, armadilhas (denominador, `complementar` da rescisão) em [[Módulo de folha e eSocial do Questor]]; padrão de query em [[Estoque e fluxo numa série a partir de datas de início e fim]]. **Depois foi reescrito sobre a view `funcionario`** (ficha atual por contrato) como base única, numa consulta-mãe só (base CTE + `json_build_object` com todas as agregações). Ganhou **métricas** (saldo de pessoal, desligamentos voluntários/involuntários por `causademissao.codigocausa`, tempo médio de casa) e **quebras** novas por **estabelecimento**, **sexo** e **faixa etária** (jovens giram muito mais — Até 24 ~30% vs 55+ ~3,5%). Endpoint `/api/folha/turnover` (tudo numa resposta).
- **Filtros avançados** (multi-select, recortam TODO o painel): estabelecimento, setor, cargo, vínculo — `/api/folha/filtros` com contagens. Vínculo = `categoria|tipovinculo` (CLT = `01|10`; o `codigocateg` vem nulo, o texto é o que vale).
- **Movimentações** (`/api/folha/movimentacoes`): lista de admitidos/desligados; cada linha abre a **ficha** do colaborador em modal (`/api/folha/funcionario`: CPF, cargo, função, setor, estab, sexo, idade, escolaridade, salário, cidade, motivo).
- **Drill em qualquer quebra** (`/api/folha/pessoas`): clicar num setor/cargo/estab/sexo/faixa/motivo/tempo abre as pessoas do grupo → ficha. Dimensões de filtro entram na base; demográficas/motivo/tempo viram predicado com as MESMAS expressões da quebra (`EXPR_FAIXA_ETARIA`/`EXPR_TENURE_FAIXA` na lib `folha-turnover`), pro recorte bater exato com o número. Componentes reusados: `FichaModal`, `PessoasTabela`, `PessoasModal`; `RotatividadeQuebra`/`Barras` ganharam `dim`+`onDrill`. Verificado end-to-end (emp 1200): quebras e drill batem número a número.
- **Quebras demográficas** (escolaridade por `grauinstr`, estado civil) drilláveis como as demais; **comparativo com o período anterior** (mesma duração, via `periodoAnterior` na própria consulta-mãe) com delta nos KPIs (turnover subiu 5,9%→14,5% no semestre); **exportação CSV** das movimentações (sep `;` + BOM p/ Excel pt-BR). Insight: Solteiro gira ~2× o Casado; Superior incompleto (estagiário) gira mais.
- **Pendente possível**: quebra por raça/cor (códigos RAIS, dado sensível — avaliar), exportação dos agregados, e o rótulo do vínculo fora do CLT ainda é cru.

### Produtividade do DP (jul/2026)

Segunda seção da Folha (`/folha/produtividade`), pedida pelo Eduardo como "o que tem no Fiscal, porém com coisas do DP": mede **o que cada pessoa do DP fez no período**, por `codigousuario` + `datahoralcto` (a auditoria embutida — mesmo padrão da Produtividade do Fiscal). Quatro trabalhos, cada um numa tabela do Questor: avisos prévios (`funcavisoprevio`), rescisões calculadas (`rescisao`), admissões (`funccontrato`), férias calculadas (`reciboferias`) — mapeamento e armadilhas em [[Módulo de folha e eSocial do Questor]]. Tela: **KPIs** (contagem dos quatro + total, com delta vs. período anterior), **ranking clicável** por colaborador (clicar filtra os detalhes por aquela pessoa), e **quatro abas** de listagem detalhada com as colunas que o DP pediu. A admissão mostra o **status eSocial do S-2200** (transmitida/pendente/não enviada) — derivado de `esocialtransacao` via `esocialdadoss2200`, `recibo` preenchido = aceito; a `xml2200evtadmissao` está vazia (staging), ver a nota de referência. Diferente da Rotatividade (uma empresa), aqui **empresa é filtro opcional** e por padrão varre todo o escopo (é o retrato do escritório) — daí a prop `empresaOpcional` nova na `ConfFilterBar`. Libs `dp-produtividade.ts`/`dp-tipos.ts`, endpoints `/api/folha/dp-produtividade` (ranking+resumo) e `/api/folha/dp-lista?tipo=` (detalhe). Validado contra o banco: julho/2026 → 55 avisos, 356 rescisões, 595 admissões (401 com recibo eSocial), 1187 férias. **Virou dashboard completo por trabalho (jul/2026)**: a pedido do Eduardo ("por colaborador, com gráficos, bem completo, tipo menus"), a tela deixou de ser um painel único com abas de lista no rodapé e passou a ter **menus no topo** (estilo das abas do módulo: `Visão geral` + uma aba por trabalho). Cada aba de trabalho é um mini-dashboard: KPIs próprios (total+delta, colaboradores, empresas atendidas, média/pessoa), gráfico **Por colaborador** (eixo principal, barra clicável isola a pessoa), gráfico **Por empresa**, **série** de evolução (diária até 92 dias, mensal acima — `generate_series` densifica os buckets vazios) e a lista detalhada agrupada por colaborador em **menu recolhível** (accordion, resolve a "altura infinita" de quem tem 500+ registros). Selecionar um colaborador (no ranking ou clicando numa barra) recorta série+empresa+lista dele em todas as abas; o ranking e o "por colaborador" ficam de fora do recorte (são a comparação entre pessoas). Duas decisões de dado que valem além daqui: a **quebra por colaborador sai do ranking já carregado** (derivar no cliente em vez de bater no banco de novo por um dado que já está na mão), e cada aba **carrega sua quebra sob demanda** (`/api/folha/dp-quebra?tipo=`, `montarQuebraDp`) em vez de um payload gigante no load. A quebra por empresa e a série respeitam o mesmo escopo/usuário das outras consultas. **Foco no funcionário da Navecon (jul/2026)**: o Eduardo reforçou que "por funcionário" aqui é o **nosso pessoal** (o operador do Questor = `codigousuario`), não o empregado da empresa-cliente cujo registro foi processado — é um painel da produtividade interna. O eixo (ranking, "por colaborador") já era esse; o ajuste tirou **de vez** o empregado do cliente da lista detalhada (as linhas mostram só empresa + datas + status) e adicionou um **seletor/busca de funcionário da Navecon** no topo que recorta o dashboard inteiro (série, por-empresa, lista) — a lista de nomes sai do ranking já carregado, sem query nova. **Abas de trabalho = só dashboard; lista numa aba só (jul/2026)**: o Eduardo achou o "registro a registro" no rodapé de cada trabalho inútil ("o Questor já tem isso"). Então a lista saiu das abas de Avisos/Rescisões/Admissões/Férias — elas ficaram **só painel** (KPIs + por colaborador + por empresa + série) — e virou **uma aba "Registros"** única, com seletor de tipo em cima (herda funcionário e período dos filtros globais). É o padrão "dashboard resume, a lista é só pra conferir quando precisa". A Visão geral ganhou mais gráfico no lugar: **donut de composição** dos quatro trabalhos e **barra empilhada** dos top colaboradores por tipo — ambos derivados do ranking/totais já carregados, sem query nova.

### Custo de Folha (jul/2026)

Terceira seção da Folha (`/folha/custo`), primeira do **Tier 1** de ideias tiradas da doc de Folha de Pagamento do Questor — a lente que faltava: o módulo tinha *pessoas* (turnover) e *produtividade do DP*, mas nunca o *dinheiro*. É o **irmão financeiro do turnover**: soma a remuneração calculada (`calculoevento`) de uma empresa no período e abre por rubrica, tipo de folha, setor/cargo/estabelecimento e mês (mesma base `funcionario`). Método e schema em [[Módulo de folha e eSocial do Questor]] (seção "Custo de folha (leitura)").

Decisões de escopo (com o Eduardo): **"custo" = proventos** (`evento.tipoevento` 1), com descontos (3) e líquido à parte; **todas as folhas de pagamento por tipo** (mensal/13º/férias/rescisão…) EXCETO adiantamento (antecipa a mensal → duplicaria), provisão (accrual) e transferência; **encargos patronais fora do MVP** (FGTS/INSS patronal não são evento no Questor — estimar por alíquota seria frágil; fase 2 mapeia a apuração patronal real). Tela: KPIs (custo/descontos/líquido/funcionários + custo médio), composição por tipo, evolução mensal, top rubricas (a memória do custo) e quebras por setor/cargo/estab. Seção nova por último na sidebar da Folha; admin vê por padrão, gate de rota no `api-secoes`.

Detalhe que vale além daqui: a rubrica é classificada **por array de códigos** (`ce.codigoevento = any($prov)`) carregados de um Map do cadastro `evento`, em vez de juntar `evento` na varredura de 5M linhas — separa a **taxonomia** (resolvida fora da query) do **somatório** (na query), corta um join pesado e desvia do join sem `codigoempresa` que o `provisoes.ts` tem latente. Compila (tsc+eslint+build); **números a validar** contra uma empresa conhecida (o `evento` como cadastro global e a classificação provento/desconto são a heurística a conferir).

### Conformidade eSocial (jul/2026)

Quarta seção da Folha (`/folha/esocial`), segunda do Tier 1: o que foi transmitido, aceito, rejeitado ou está pendente no eSocial da empresa no período. Fonte `esocialtransacao` (regra em [[Módulo de folha e eSocial do Questor]]: `recibo` preenchido = aceito; sem recibo + `status` 13 = rejeitado; sem recibo, sem rejeição = pendente). Duas lentes: **panorama por tipo de evento** (S-2200, S-2299, S-1200… × situação, com barra de proporção aceito/pendente/rejeitado, direto de `esocialtransacao` agrupado por `evento`) e as **pendências obrigatórias** que o DP tem de resolver — admissões sem S-2200 aceito e rescisões sem S-2299 aceito, ligando contrato→transação pela `esocialdadoss<NNNN>` (última por `datahoralcto`), o mesmo padrão que a Produtividade do DP já usava para o status do S-2200. KPIs (transmitidos/aceitos/pendentes/rejeitados), as duas listas de pendência em destaque e a tabela por evento. Compila (tsc+eslint+build); **a validar no banco**: `esocialdadoss2299` (rescisão) e `status=13` (rejeitado) — a tela expõe as pendências e o resultado por evento, então um erro se denuncia.

### Controle de Férias (jul/2026)

Quinta seção da Folha (`/folha/ferias`), **fecha o Tier 1**: quem tem férias vencidas (risco de pagar em DOBRO) ou a vencer. Como o Questor não expõe (no schema conhecido) uma tabela de saldo de período aquisitivo, o cálculo **deriva** os períodos de `dataadm` (de 12 em 12 meses) e cruza com os aquisitivos já gozados em `reciboferias.datainicial`; período completo há > 12 meses sem recibo = vencido. Regra CLT: aquisitivo (12m) dá 30 dias, concessivo (mais 12m) é o prazo de conceder, senão dobro — limite = fim do aquisitivo + 12 meses. Só empregado CLT (`categoria='01'`), foto na data de referência (fim do período do filtro). KPIs (ativos, com vencidas, a vencer em 120d, total de períodos vencidos = o passivo), lista por criticidade e toggle "só vencidas e a vencer". **Ressalva na própria tela** (é aproximação): ignora afastamentos, faltas e férias parciais/coletivas — [[Deixar o método da conferência visível quando o SQL não foi validado]]. Método e a lacuna da tabela nativa em [[Módulo de folha e eSocial do Questor]].

Com isso o **Tier 1 do módulo Folha está completo** (Custo de Folha, Conformidade eSocial, Controle de Férias) — os três com números a validar contra uma empresa conhecida no banco read-only. As três seções nasceram em branches próprias (`feat/folha-custo`, `feat/folha-esocial`, `feat/folha-ferias`), empilhadas sobre a `feat/contabil-auditoria`.

### Relatório Post Mortem do DP (ago/2026)

Sexta seção da Folha e a primeira em que o usuário **preenche** (as outras leem o Questor): um **Relatório Post Mortem** de incidente do DP, pedido pelo Eduardo a partir de um modelo Word. Nessa entrega o módulo **Folha virou "DP"** no launcher — só o rótulo em `modulos.ts`; id e rotas seguem `folha` pra não migrar permissão nem quebrar URL (Folha e DP são a mesma coisa pro Eduardo). Branch `feat/dp-post-mortem`.

O analista logado preenche de forma web; o nº do relatório é **sequencial**, alocado no ENVIO (uma `sequence`, sem corrida entre envios). Campos de escolha: **criticidade** (4 níveis do Word) e **grupo** (dropdown de `empresa_grupo`, os grupos do admin).

- **Posse por seção + linha**: como a permissão do Hub é binária, "analista vê os SEUS / gestor vê TODOS" virou **duas seções** (`post-mortem` e `post-mortem-gestao`) + recorte por `autor` no servidor (na rota e na página), em vez de um nível view/edit — [[Posse numa permissão binária é duas seções e recorte por linha]].
- **Modelagem**: o que vira indicador/filtro (nº, criticidade, grupo, datas, status) é **coluna**; a narrativa e as tabelas repetíveis (linha do tempo, 5 porquês, ações) vão em **jsonb** — [[Campo que vira indicador é coluna, o resto do documento é jsonb]]. Migration 019.
- **`apiRoute`** passou a repassar o `ctx` (params) ao handler, pra a rota dinâmica `[id]` autenticada ler o segmento sem sair do gate central.
- O shell da Folha **pula o filtro de empresa** nessas seções (são self-contained, como as telas internas do RH).

Verificado: tsc/eslint/build e round-trip do SQL contra o banco (rascunho → salvar jsonb → enviar/sequence → ler com joins). **Pendente (v2)**: extração + IA (reuso [[Coleta determinística, LLM só interpreta]] + [[Censurar a identificação antes de mandar pro LLM externo]]) + indicadores na tela de Gestão.

### Rescisões a pagar (ago/2026)

Sétima seção da Folha (`/folha/rescisoes`), a segunda **operacional** do DP: a **fila das rescisões a pagar** com o prazo legal (CLT art. 477 §6: verbas em até 10 dias do fim do contrato, configurável). Pega os desligamentos do período (`funccontrato.datadem`, só CLT `categoria='01'`, ignorando transferências), cruza com a rescisão calculada (`rescisao`, causa via `causademissao`) e a folha de rescisão do Questor, e classifica cada uma por urgência contra o prazo: **vencida**, **vence em breve** (dentro da antecedência), **no prazo**, **paga**. KPIs (pendentes, vencidas, a vencer, pagas), toggle "só pendentes", ação "marcar paga". Empresa **opcional** (retrato do escritório, como a Produtividade do DP — reusa `empresaOpcional` na `ConfFilterBar`), escopo pela sessão.

- **A rescisão só fecha por ato explícito** — a marcação manual "paga" (com data + observação), gravada no banco do app. O sinal do Questor (folha calculada, `periodocalculo.datapgto`) aparece como coluna de apoio mas **não** resolve sozinho, porque `datapgto` é a data prevista (some antes do pagamento): [[Uma pendência de prazo fecha por ato explícito, não por sinal inferido]]. O override mora no app chaveado por (empresa, contrato) do Questor — [[Sobre fonte read-only, o editável mora no seu banco chaveado pela identidade dela]].
- **Aviso por e-mail** = um cron diário (`/api/folha/cron/rescisoes`, público com `RH_CRON_SECRET`, igual aos do RH) que varre as pendentes dos últimos 180 dias e manda UM e-mail-resumo com os avisos novos: antecedência = aviso único (slot fixo), atrasada = lembrete diário (slot = dias negativos). Idempotente por `(empresa, contrato, slot)` no log `rescisao_aviso`, mesmo mecanismo dos lembretes de experiência — [[Recorrência guarda a receita e o próximo disparo, não N ocorrências futuras]]. Destinatários são o time do DP, cadastrados no app (o Questor não tem e-mail de ninguém).
- **Migration 026**: `rescisao_config` (prazo + antecedência, singleton), `rescisao_destinatario`, `rescisao_resolvida` (override), `rescisao_aviso` (log). Libs `rescisoes.ts`/`rescisoes-tipos.ts`. Schema da rescisão→folha 60→`datapgto` em [[Módulo de folha e eSocial do Questor]].

Verificado: tsc/eslint/`next build` e migration aplicada no banco do app. **A validar no host** (o Questor de produção não é acessível do dev — timeout): abrir a tela e rodar o cron uma vez para o smoke test da query. Branch `feat/rescisoes`, merge ff na `main`.

## Módulo RH (interno da Navecon — jul/2026)

Quarto módulo, o primeiro **interno** (não é sobre os clientes do escritório, e
sim sobre o pessoal da própria Navecon). Escopo FIXO nas **empresas Navecon do
Questor**: NAVECON (`codigoempresa 1`, o próprio escritório), FOUR
(`codigoempresa 888`) e **FINAVE (`codigoempresa 746`, incluída jul/2026)** — mesma
Navecon, CNPJs distintos, mesmos departamentos. O escopo é forçado
(`EMPRESAS_RH = [1, 888, 746]` em `src/lib/rh.ts`), **ignorando o grupo de empresas
da sessão** de propósito: o grupo padrão do Hub é "Todas menos NAVECON", que
esconderia justo a empresa 1. O gate do módulo é ter as seções do RH; o dado sempre
se limita a essas empresas. Como TUDO deriva de `EMPRESAS_RH`/`nomeEmpresaRh`,
incluir uma empresa é uma linha só (diretório, gestores, experiência, envios e
rotatividade propagam sozinhos) — o resto do trabalho foi só corrigir textos que
diziam "as duas empresas". Reusa a casca inteira (uma linha em `modulos.ts`,
`rh-secoes.ts`, gate por seção).

- **Diretório** (`/rh/diretorio`): funcionários ativos das empresas do RH, filtro
  por empresa/setor + busca e contagem por empresa (tudo no cliente — dataset
  pequeno); ficha em modal. A ficha reusa a QUERY da Folha extraída para
  `funcionario-ficha.ts`, cada módulo servindo pela sua rota — de novo
  [[O que dois módulos compartilham é a query, não a rota]]. **Editável (ago/2026):**
  o Questor vinha bagunçado e sem os PJ, então o Diretório ganhou uma camada
  gravável no app-db — corrigir qualquer campo do funcionário (overlay), incluir
  pessoas PJ (aba própria) e renomear/criar setores. O Questor segue intocado;
  padrão em [[O que o Questor não dá mora no app-db chaveado pela identidade dele]].
  PJ também entra nos formulários "sobre um colaborador" (roteado por setor→gestor).
- **Gestores** (`/rh/gestores`): cadastro (banco do app) de supervisores/
  coordenadores POR SETOR do organograma, N por setor (nome + e-mail + papel). É
  o que o Questor não tem — ver [[Módulo de folha e eSocial do Questor]] (buraco
  de supervisor/e-mail). Alimenta o destinatário do formulário de experiência.
- **Formulários** (`/rh/formularios`): construtor tipo Google Forms (jul/2026) —
  a RH monta formulários com campos de **escrever**, **marcação** (uma/várias
  opções), **nota** (escala) e **pontuação**, cada um uma pergunta, com preview ao
  vivo. `config` jsonb por tipo guarda opções/escala/intervalo (o form evolui sem
  migration). Um campo `selecao_unica` pode ser marcado como **decisão** — vira o
  destaque nos painéis, no lugar da antiga "recomendação" fixa. Vários formulários,
  status rascunho/ativo/arquivado. Padrão em [[Formulário montado pelo usuário — a
  definição no banco dirige renderer e validação]].
- **Experiência** (`/rh/experiencia`): o coração. Contrato CLT = **45 + 45 dias**,
  dois marcos (45/90). Os critérios **deixaram de ser fixos no código**: a RH
  **liga um formulário ativo a cada marco** e define a **antecedência** do aviso
  (padrão **7 dias** antes) na própria tela (`rh_experiencia_config`). O painel
  projeta os marcos em curso com status (aguardando · respondido · **atraso**). O
  **job diário** (`/api/rh/cron/experiencia`, `RH_CRON_SECRET`) dispara **um**
  lembrete na antecedência configurada + um aviso de **atraso** (antes eram
  D-15/10/5/1), idempotente por slot. E-mail para todos os gestores do setor via
  SMTP (`nodemailer`, driver de log quando não configurado).
- **Formulário público** (`/f/[token]`, unificado): o gestor responde **sem
  logar**, por token opaco. Renderizado **dinamicamente** a partir da definição do
  formulário; validação de verdade **no servidor**. Serve tanto a experiência
  quanto as campanhas; `/experiencia/[token]` virou **redirect** (links antigos
  valem). Padrão em [[Formulário público por token opaco fica fora do gate de sessão]].
- **Campanhas de envio** (aba **Envios** em Formulários): a RH envia qualquer
  formulário ativo para gestores (**todos ou alguns**) e/ou **e-mails avulsos**,
  **agora ou agendado**. Cada destinatário recebe token próprio, responde uma vez,
  e a RH acompanha as respostas. Job `/api/rh/cron/envios` (mesmo segredo) dispara
  as agendadas. **Sobre um colaborador (jul/2026)**: além do broadcast, a campanha
  pode ser uma avaliação SOBRE colaboradores específicos — a RH marca vários, cada
  um vira uma avaliação enviada aos gestores do departamento dele (`classiforgan`)
  e aceita UMA resposta (o primeiro gestor que responde fecha), igual à experiência.
  Reusa `envio_destinatario` com colunas de colaborador + `email` nullable (os
  destinatários reais saem no disparo, resolvidos do setor); colaborador sem gestor
  no depto é ignorado e reportado; índice único evita repetir o mesmo colaborador; o
  form público mostra o contexto do colaborador. Padrão em [[Uma resposta canônica
  de um grupo é um token compartilhado]].
- **Rotatividade** (`/rh/rotatividade`): o turnover das duas empresas, período +
  empresa, reusando a Folha inteira. A consulta-mãe saiu para
  `folha-turnover-query.ts` (turnover + movimentações + pessoas) e `construirBase`
  ganhou "empresa forçada": a Folha usa o escopo da sessão, o RH força NAVECON/
  FOUR — a query é compartilhada, a rota não. Como o painel mistura as duas
  empresas, `FolhaMovimentacao` passou a carregar `codigoempresa` e a ficha abre
  pela empresa da linha (contrato é PK por empresa, colide entre as duas).

Migrations do RH: `008_rh.sql` (gestores + experiência: `rh_setor_gestor`,
`rh_experiencia`, `rh_experiencia_resposta`, `rh_experiencia_lembrete`),
`011_formularios.sql` (`formulario`/`formulario_campo`), `012_experiencia_formulario.sql`
(`rh_experiencia_config` por marco, `formulario_id` na experiência, recomendação
relaxada p/ nullable), `013_envios.sql` (`envio`/`envio_destinatario`),
`014_envio_sobre_colaborador.sql` (colaborador em `envio_destinatario`, `email`
nullable, índice único por colaborador), `024_pj_experiencia.sql` (`tem_experiencia`
no PJ) e `025_envio_regra.sql` (`envio_regra`, regra de envio automático). As respostas
moram junto do envio que gerou o token (experiência ou campanha); o formulário é só
a definição. Verificado ponta a ponta (form dinâmico → token → resposta → status;
cron de campanha em modo log, sem e-mail real). **SMTP real** configurado (Gmail
`noreply.navecon@gmail.com`, senha de app no `.env` gitignored); `RH_CRON_SECRET`
setado. O disparo automático **deixou de depender do cron do host**: um serviço
`scheduler` no compose bate as rotas sozinho (ver a leva de ago/2026 abaixo). Falta só
conceder a seção `formularios` ao cargo da gestora no /admin. Branch `feat/rh-formularios`.

### Formulários, envio automático e PJ na experiência (ago/2026)

Segunda leva do RH, deixando o envio de formulários **robusto e automatizável**:

- **PJ na experiência**: prestador PJ (cadastrado à mão) ganhou a caixinha
  `tem_experiencia` (cadastro e ficha). Marcado, entra na **mesma trilha** dos CLT —
  marcos 45/90 a partir da data de início, via o contrato sintético
  (`PJ_CONTRATO_OFFSET + id`), indo aos gestores do setor. Reusou tudo: só uni o PJ à
  consulta de contratos e ao `buscarUmContrato` do reenvio.
- **Excluir avaliação de experiência**: antes só dava pra ver as respostas (envio de
  teste ficava preso). `DELETE /api/rh/experiencia?id=` + lixeira no painel; o `on
  delete cascade` leva resposta e lembretes. Como o painel projeta do contrato, a
  pessoa volta a aparecer como pendente.
- **Disparo automático garantido**: o código dos jobs existia, mas nada os agendava.
  Novo serviço `scheduler` no compose (imagem do app, entrypoint próprio sem
  migrations) bate `/api/rh/cron/envios` (15 min) e `/api/rh/cron/experiencia` (1x/dia).
  Técnica em [[Agenda recorrente é um serviço do compose, não um crontab do host]].
- **Modal de envio com dois eixos**: o toggle "Para gestores / Sobre colaboradores"
  misturava *quem responde* com *sobre quem é o formulário*. Virou dois seletores —
  **Quem responde** (gestores / colaboradores / e-mails avulsos) e **Sobre quem**
  (genérico / um colaborador), este só quando o gestor responde. Novo caminho: enviar
  **direto ao colaborador**. Como o Questor não tem e-mail, o campo passou a viver no
  **overlay do Diretório** (e no PJ) — aplicação de
  [[Sobre fonte read-only, o editável mora no seu banco chaveado pela identidade dela]];
  quem não tem e-mail é sinalizado e ignorado.
- **Envio automático recorrente**: nova aba **Automático** com regras (`envio_regra`)
  que enviam um formulário sozinhas — mensal no dia X ou a cada N dias, para gestores
  ou colaboradores, alvo todos / por setor / colaboradores específicos. O job resolve o
  público no disparo (reusa `criarEnvio`) e reprograma; roda dentro de
  `/api/rh/cron/envios`. Técnica em
  [[Regra de envio recorrente materializa uma campanha e reprograma]], sobre o princípio
  [[Recorrência guarda a receita e o próximo disparo, não N ocorrências futuras]] — que
  a experiência já seguia (materializa a linha só ao entrar na janela). Verificado o SQL
  do motor contra o banco (CTE insert-returning, jsonb do alvo, seleção das vencidas e
  reprogramação), build/lint/tsc limpos.

### Canal de denúncia + Clima (ago/2026)

Duas seções pedidas pela RH: um **canal de denúncia anônimo** (assédio /
irregularidades, atende a Lei 14.457/2022) e uma **avaliação anônima da empresa**
(clima), ambas com painel de gestão. Branch `feat/canal-rh`, migration 018.

- **Acesso é por LINK PÚBLICO aberto** (sem login, sem token por pessoa),
  diferente da experiência (token por destinatário, responde uma vez, exige nome).
  Forçar denúncia/clima no pipeline `/f/[token]` distorceria aquele fluxo, então
  ganharam **rotas públicas próprias** (`/denuncia`, `/denuncia/acompanhar`,
  `/clima/[slug]`) reusando só as peças finas (validação, hash, componentes) — de
  novo a exceção cirúrgica de [[Formulário público por token opaco fica fora do
  gate de sessão]]. **Pegadinha do proxy**: como as rotas eram novas e fora da
  lista de exceções do `src/proxy.ts`, o redirect otimista mandava o visitante
  anônimo pro `/login` — precisou liberar `denuncia`/`clima` no matcher (sem barra,
  então `/rh/denuncias` e `/rh/clima`, que começam com `rh`, seguem protegidos).
- **Anonimato por desenho** (o aprendizado que virou nota):
  [[Canal anônimo não guarda quem, e o retorno é um segredo do denunciante]].
  Nenhuma tabela grava IP/identidade; o denunciante acompanha por **protocolo +
  senha** (senha só como hash scrypt, irrecuperável); no clima, resposta sem nome.
  O dashboard suprime recorte por setor abaixo de N respostas pra não deanonimizar.
- **Denúncia** (`/rh/denuncias`): fila com filtro por status/assunto, mini-dashboard
  (total, aguardando RH, em análise, tempo de 1ª resposta) e painel de tratativa
  (relato + thread com o denunciante anônimo + responder + mudar status).
- **Clima** (`/rh/clima`): rodadas (criar/abrir/fechar, link próprio por slug) e
  dashboard — **eNPS** (0–10, promotores−detratores) + breakdown, distribuição
  colorida por faixa, média por tema (temas configuráveis em jsonb na rodada),
  tendência entre rodadas, recorte por setor e comentários. Uma rodada seed
  (`clima-2026`) já nasce aberta.
- Gestão gateada pelo `apiRoute` (seções `rh/denuncias`, `rh/clima` no
  `api-secoes`; entram sozinhas no /admin por virem do catálogo de código). A
  caixa de link público monta a URL via `useSyncExternalStore` pra não brigar com
  a hidratação (parente de [[Portal condicional dispensa o flag de montagem]]).
- Verificado ponta a ponta: build + tsc + eslint limpos; fluxos públicos por curl
  (criar denúncia → protocolo+senha → consultar; senha errada → 404; clima
  responde e valida faixa; mensagem do denunciante entra na thread); rotas de
  gestão dão 401 sem sessão e a página 307→login; as queries de dashboard rodadas
  direto no banco. **Pendente** (precisa de sessão real, não dá pra dirigir daqui):
  conceder as seções `denuncias`/`clima` ao cargo da gestora no /admin, e o
  passeio autenticado pelos dois painéis. QR do link ficou de fora (sem lib de QR;
  hoje é copiar/abrir o link) — follow-up fácil.

## Filial (estabelecimento) no filtro — jul/2026

O sistema filtrava só por **empresa** (`codigoempresa`); no Questor a empresa
tem N **filiais** (`codigoestab`), e o `buildWhere` (funil de todo o Fiscal/
Contábil) somava todas as filiais cegamente — sem como isolar. **163 empresas
têm mais de uma filial** (extremo: a 863 com 31). Toda tabela do funil (nota,
item, `lctoctb`) tem `codigoestab`, então dá pra recortar.

Adicionado `estabs: number[]` ao filtro (vazio = todas = consolidado); o
`buildWhere` aplica `codigoestab = any(...)`. Um `FilialDropdown` (multi-seleção)
aparece só quando há **uma** empresa em escopo e ela tem >1 filial — `codigoestab`
não é comparável entre empresas (com várias selecionadas, some; a princípio nada
seleciona mais de uma). Endpoint `/api/empresas/estabs`.

**Fiscal saiu 100% na hora** (tudo passa pelo buildWhere). **Contábil precisou
costurar à mão**: Conferência e Balancete têm **query própria** (não usam
buildWhere). Feito e verificado:
- **Conferência**: recorte nos scans de FATO (notas + consolidação MOV); os
  scans "por chave" (lctoctb/itens) seguem as notas já filtradas.
- **Balancete** (o delicado): `estabs` em TODOS os scans dos DOIS lados do
  espelho — motor (notas, param dinâmico), lado real (lctoctb FI), `pendentesNfse`
  (só as pendentes; a previsão de conta fica por empresa) e os 4 drills
  (calibração `observadas`/`lancadas` passa a ser por filial, pro drill bater com
  a célula). Verificado na 863 (30 filiais): partição do lado real bate exata com
  o consolidado (R$ 87,8 MI). A calibração vira filial-scoped — correto, mas é o
  que torna o filtro delicado: filtrar só um lado desalinha.

"Conferência de Contas" não é rota — é lib pura (`divergencias.ts`), coberta pela
Conferência. A Folha já tinha filtro de estabelecimento próprio (por rótulo).
Lição virou [[Filtro transversal só é honesto se todo o funil o honra]].
Branch `feat/filial` (a partir da `main`).

## Stack e contexto técnico

- Next.js 16 (App Router) + React 19, TypeScript, Tailwind v4.
- React Query pra data-fetching com `keepPreviousData` (refetch sem piscar).
- Recharts pros gráficos. Paleta seguindo [[Validar paleta de gráficos antes de escolher cores]].
- `pg` com pool, conexão **somente leitura** (`default_transaction_read_only=on`, `statement_timeout` 60s) — BI nunca altera o Questor.
- **Segundo banco, próprio e gravável** (`src/lib/app-db.ts`, pool separado): Postgres 17 em Docker Compose na porta **5022** (espelho de 4022), migrations SQL versionadas em `migrations/` com runner próprio (`npm run migrate`). Guarda os overrides do plano de contabilização e é onde vão morar login/usuários/permissões. Subir com `npm run db:up && npm run migrate`. O Questor continua intocado.
- Rodar em dev: `npm run db:up && npm run migrate && npm run dev`.
- **Deploy na rede (jul/2026)**: tudo em Docker. `docker compose up -d --build` no computador que hospeda sobe app + banco do BI e deixa acessível em `http://<ip>:4022` para qualquer máquina da rede. Dockerfile multi-estágio com `output: "standalone"` no `next.config.ts` (imagem mínima; `public` e `.next/static` precisam ser copiados à mão, o standalone não os inclui). As **migrations rodam no boot** via `docker-entrypoint.sh`, então não há passo manual. Detalhe que pega: o `APP_DB_URL` do `.env` aponta `localhost:5022` (dev) e quebraria dentro do compose — por isso o serviço `app` reexporta `APP_DB_URL` em `environment:`, que tem precedência sobre `env_file:`. O Postgres próprio é publicado só em `127.0.0.1:5022`, sem exposição à rede.

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
- [[Contas bancárias e layout de contabilização no Questor]]

Gerais de dev (continuação):
- [[Ler extrato bancário em PDF]]
- [[Armadilhas de child_process no Node]]
- [[Recorrência guarda a receita e o próximo disparo, não N ocorrências futuras]] (princípio)
- [[Regra de envio recorrente materializa uma campanha e reprograma]]
- [[Agenda recorrente é um serviço do compose, não um crontab do host]]

Gerais de dev:
- [[Canal anônimo não guarda quem, e o retorno é um segredo do denunciante]]
- [[Permissão composta por papéis somados, não exceção por usuário]]
- [[Agregar antes de juntar em tabelas gigantes no Postgres]]
- [[router.replace do Next falha no build de produção]]
- [[Validar paleta de gráficos antes de escolher cores]]
- [[Versão é corte deliberado em SemVer, não efeito de cada merge]]
- [[Para alimentar o ERP, gere o arquivo de importação dele]]
- [[De-para determinístico com override que vira aprendizado]]

Design (reutilizável em outros projetos — ver [[Design]]):
- [[Sistema de cores e tema do dashboard]]
- [[Padrões de componentes de dashboard]]
- [[Acento da interface é um token separado da cor de dado]]

### Repaginação visual — Índigo/Vidro (ago/2026)

Redesign transversal da cara do sistema, a pedido do Eduardo ("muito mais moderno e bonito, em tudo") mais a limpeza dos textos de tela. Duas frentes:

- **Nova identidade de interface** com acento de marca **índigo→violeta**, separado das cores de dado (azul=entrada, verde=saída, intactas) — a separação virou técnica: [[Acento da interface é um token separado da cor de dado]]. Fundação nos tokens do `globals.css`: chrome de vidro (`backdrop-filter`) nas sidebars, fundo com halo de acento, sombras em camadas, cantos mais suaves, **anel de foco** global e KPIs/modais redesenhados. Quase tudo saiu de token + poucas classes centrais, não arquivo a arquivo — reforça [[Token semântico em vez de valor literal]] e [[Classes de componente vão em @layer components no Tailwind]].
- **Limpeza da prosa da UI**: removidos ~90 textos explicativos (banners "como funciona", fórmulas, legendas "clique para ver…", subtítulos redundantes), preservando rótulos curtos e os avisos legais das telas públicas. Decisão do Eduardo: interface de produção não ensina com parágrafo.

Feito na branch `feat/redesign-visual`; build de produção limpo (161 rotas). A estética índigo/vidro é **deste projeto** — o dialeto de [[Design]] segue sendo o "Aurora Glass"; o que se reusa é o princípio, não a cara ([[Estética é por projeto, princípio de design é que se reusa]]).

### Base de primitivos + glass em camadas (ago/2026)

Depois da repaginação, o gargalo não era mais a estética e sim a **falta de base
reaproveitável**: de 212 arquivos `.tsx`, 79 aplicavam `.card` na mão e 61 tinham botão
solto, mas não existia componente `Button` (havia três "primário" concorrentes —
gradiente, `bg-accent`, `bg-ink`) nem `Card`/`Badge`/`Table`/`StatTile`. Criada a camada
`src/components/ui/` — `Button`/`IconButton`, `Card`, `EmptyState`, `StatTile`, `Badge`,
`Segmented`, `Table` (Thead/Th/Tr/Td) — cada um com `className` overridável pelo helper
`cn` (clsx + tailwind-merge). A casca (sidebar, launcher, admin, modais) e o módulo
**Fiscal** entraram primeiro como vitrine (branch `feat/base-primitivos-glass`); em
seguida **todos os outros módulos** — Contábil, Folha, RH, Admin, Denúncia/Clima, Perfil,
Login — foram migrados sobre a base (branch `feat/primitivos-modulos-restantes`, saldo
−313 linhas: a repetição de classe deu lugar aos primitivos). A varredura em massa dos
módulos restantes foi paralelizada em subagentes por diretório disjunto, com a spec das
APIs dos primitivos e a regra de preservar tabelas de chrome própria e toggles de valor
numérico. De quebra, corrigiu o token quebrado `text-warn` (sem `--color-warn` no tema)
para `warning` em vários lugares. Build de produção limpo em cada corte.

O glass também virou **dois níveis explícitos**: `.glass-chrome` (arejada) e
`.glass-panel` (overlay opaco e legível), com blur/saturação mais macOS e uma curva
`--ease-spring` no hover. Aprendizados que subiram pra base (o projeto só linka):

- [[Primitiva de botão fecha o tamanho e abre só a variante]] — agora com 2º caso.
- [[A classe do chamador só vence a do primitivo com tailwind-merge]] — o helper `cn`.
- [[Vidro flutuante precisa de superfície mais opaca que a chrome]] — chrome vs panel.

## Próximos passos

- [x] Login e usuários (jul/2026) — feito. Autenticação por email/senha, sessão opaca no banco, 3 eixos (módulo/seção/empresa com grupos), área `/admin`. Ver "Arquitetura de módulos e permissão" acima.
- [ ] Folha: recorte por categoria de vínculo na Rotatividade (só CLT), quando o banco estiver acessível e os `codigocateg` confirmados; outras telas de folha (custo de pessoal, headcount, provisões).
- [ ] Módulo Patrimônio (ainda "em breve" no launcher) — reusar o padrão de módulo/seção/abas.
- [ ] Talvez repensar grupos de empresas (hoje só localStorage; poderia ser compartilhado entre máquinas).
- [ ] Possíveis análises futuras: mapa por UF, exportação Excel.

## Links

- Mapa: [[Projetos]]
