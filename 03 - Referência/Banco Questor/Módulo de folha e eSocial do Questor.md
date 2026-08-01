---
tags: [tipo/atomica, camada/referencia, dev/backend, banco/questor, sql]
criado: 2026-07-18
---

# Módulo de folha e eSocial do Questor

> A folha separa **pessoa** (`funcpessoa`) de **vínculo** (`funccontrato`, por empresa); os dados que mudam (salário, cargo, lotação) ficam em tabelas de vigência por `datainicial`; e o resultado calculado da folha é `calculoevento` (rubrica × funcionário × período). A view `funcionario` já junta tudo.

## Núcleo: pessoa × vínculo

- **`funcpessoa`** — a **pessoa física** (96 colunas): `codigofuncpessoa` PK; `nomefunc`, `cpffunc`, `pisfunc`, endereço, nascimento, documentos. **Global**, sem `codigoempresa` — a mesma pessoa pode trabalhar em várias empresas.
- **`funccontrato`** — o **vínculo empregatício** por empresa: PK `(codigoempresa, codigofunccontr)`; `codigofuncpessoa` (FK → pessoa), `dataadm`, `datadem`, `categoria`/`codigocateg`, `tipovinculo`, `codigosit` (situação). Uma pessoa = vários contratos.
- ~19 mil pessoas / ~21 mil contratos na base.

## Vigências ("último registro por data")

Dados que mudam no tempo ficam em tabelas com `datainicial` (PK inclui a data); o **estado atual é o de maior `datainicial`**:

- `funcsalario` — salário (`valorsal`, `tiposalario`).
- `funccargo` — cargo/função (`codigocargo` → `cargo`; `codigofuncao` → `funcao`).
- `funclocal` — lotação por vigência: PK `(codigoempresa, codigofunccontr, datatransf)`; aponta `codigoestab` + `classiforgan` (o **organograma/setor**) e dados CAGED/RAIS. A vigência aqui é `datatransf` (data da transferência), não `datainicial`; lotação atual = maior `datatransf`.
- `funcescala` (jornada/horário — a vigência liga o contrato à `escala`), `funcctps`, `funclegais`, `funcsindicato`, `funcadicional` (insalubridade/periculosidade), etc. **`escala` é cadastro GLOBAL** (PK só `codigoescala`, sem `codigoempresa`); `descrescala` é o **horário legível** (ex.: "08:00 às 12:00/14:00 às 18:00"), com `turno` e `cargahorsemanal`. A view `funcionario` já traz `codigoescala` (e `descrtipojornada`, que é a jornada por extenso — texto longo, ruim de agrupar; para filtrar/quebrar por "horário", use `escala.descrescala`).
- `cargo` (`descrcargo`, `cbo` → `cbo`) e `funcao` (`descrfuncao`) são cadastros globais.

## Rotatividade (turnover): admissão, desligamento e efetivo

Tudo sai de `funccontrato`, no nível de **contrato**:

- **Admissões** no período = contratos com `dataadm` dentro do intervalo.
- **Desligamentos** no período = contratos com `datadem` dentro do intervalo.
- **Efetivo numa data D** (headcount) = contratos admitidos até D e ainda não desligados: `dataadm <= D and (datadem is null or datadem >= D)`. É estoque, consequência do fluxo — mesma ideia de [[Balancete é movimento do período, saldo é consequência]].
- **Turnover** = ((admissões + desligamentos) / 2) ÷ colaboradores ativos × 100. Existem variantes do denominador — a fórmula "de livro" usa **efetivo médio** ((início+fim)/2), mas o relatório de RH que o escritório usa como referência usa **colaboradores ativos = efetivo no FIM** do intervalo (verificado batendo número a número: setor com 7 ativos, 1 adm e 1 dem → 14,29% = (1+1)/2/7). Alinhar ao denominador do DP, senão os números divergem do que ele confere.
- **Quebra por setor (organograma)**: atribui cada contrato à sua lotação vigente (`funclocal` → `classiforgan` + `codigoestab`) e junta em `organograma (codigoempresa, codigoestab, classiforgan)` pra pegar `descrorgan` (o nome do setor, ex.: TINTURARIA MALHA). Some por setor os mesmos ativos/adm/dem.
- **Quebra por cargo**: mesmo padrão, pela vigência de `funccargo (codigoempresa, codigofunccontr, datainicial)` — cargo atual = maior `datainicial`; `codigocargo` → `cargo.descrcargo` (`codigofuncao` → `funcao.descrfuncao`).
- **Motivo do desligamento**: mora em `rescisao (codigoempresa, codigofunccontr, complementar)`, coluna `codigocausa` → `causademissao.descrcausa` (ex.: "Demissão Sem Justa Causa", "Inic.Empregado S/Justa Causa"/pedido, "Fim do Contrato"). Armadilha: **`complementar` não é sempre 0** — nesta base todas as rescisões têm `complementar = 1`; não fixe `= 0` (zera o join), pegue a de menor `complementar` por contrato (`distinct on … order by complementar`).
- **Tempo de casa** de quem saiu = `datadem - dataadm` (dias), em faixas.
- **Voluntário × involuntário**: pela `causademissao.codigocausa`. Iniciativa do **empregado** (voluntário): 3, 4 (pedido "Inic.Empregado S/Justa Causa"), 7, 14. Iniciativa do **empregador** (involuntário): 1, 2 (sem justa causa), 5, 11, 13. O resto (fim de contrato 12/16, falecimento, aposentadoria, extinção) é "outros".

### A view `funcionario` é a base pronta de qualquer relatório de folha

A view `funcionario` (282 colunas) já é a **ficha atual por contrato** com as vigências resolvidas: junta `funccontrato` + `funcpessoa` + o registro mais recente de salário/cargo/lotação. Para agregados de RH, montar a base sobre ela sai muito mais barato que rejuntar as `func*` na mão. Traz de graça: `dataadm`/`datadem`, `categoria`/`tipovinculo`, `codigocargo`, `classiforgan` + `codigoestab` (lotação), `valorsal`/`tiposalario`/`descrsal`, e a **demografia** — `sexo` (1 M / 2 F), `datanasc` (→ idade via `age()`), `grauinstr`, `estadocivil`, `racacor`, `nomefunc`, `cpffunc`, endereço (`siglaestado`/`codigomunic` → `municipio`). Só os **nomes** não vêm resolvidos: `descrorgan` (`organograma`), `descrcargo` (`cargo`), `descrfuncao` (`funcao`), `apelidoestab`/`nomeestab` (`estab`) — junte on demand.

Códigos demográficos (não têm tabela de domínio no banco — decodifique no código): **`grauinstr`** = eSocial tabela 18 (1 Analfabeto … 5 Fundamental completo, 6 Médio incompleto, 7 Médio completo, 8 Superior incompleto, 9 Superior completo, 10 Pós, 11 Mestrado, 12 Doutorado). **`estadocivil`** 1 Solteiro, 2 Casado, 3 Divorciado, 4 Separado, 5 Viúvo, 6 União estável. **`racacor`** usa os códigos **RAIS** (não eSocial): 1 Indígena, 2 Branca, 4 Preta, 6 Amarela, 8 Parda, 9 Não informada.

Conta-se por **contrato**, não por pessoa (uma pessoa com dois vínculos conta dois). Armadilha: **transferência** entre estabelecimentos da mesma empresa gera par desligamento+admissão que infla o índice. Recorte de vínculo: nesta base o `codigocateg` (smallint) vem **nulo** — quem tem valor é `funccontrato.categoria` (varchar, ex.: `'01'` empregado, `'07'`, `'11'`) e `tipovinculo` (varchar, ex.: `'10'` CLT, `'80'` diretor); use esses pra filtrar CLT. E `codigosit` é status cadastral (=1 pra todos), **não** serve de ativo/inativo — o vivo/desligado é `datadem`. A série mensal sai do padrão [[Estoque e fluxo numa série a partir de datas de início e fim]].

## Cálculo da folha

- **`periodocalculo`** — o período/competência: PK `(codigoempresa, codigopercalculo)`; `compet` (competência), `datainicialfolha`/`datafinalfolha`, `datapgto`, `codigotipocalc` → `tipocalculo`, `fechado`.
- **`tipocalculo`** (tipo de folha): `1` Mensal, `8` Adiantamento salarial, `20-23` 13º salário, `40` Intermitente, `50/52` Férias (normais/coletivas), `60` Rescisão, `70/71` Provisão (13º/férias), `10/12` Distribuição/PLR, `6` Dissídio, `80` Transferência.
- **`funcpercalculo`** (~273k) — liga um contrato a um período calculado.
- **`calculoevento`** (~5,1M) — **o resultado da folha**: PK `(codigoempresa, codigopercalculo, codigofunccontr, codigoevento)`; `referevento` (referência: horas/dias/%), `valorevento` (R$), `baseevento`. É "quanto cada rubrica rendeu para cada funcionário em cada período".

### Custo de folha (leitura) — `calculoevento` × `evento`

Para "quanto a folha custou no período": `sum(ce.valorevento)` de `calculoevento`, juntando `periodocalculo` (recorte pelo **fim da folha** `datafinalfolha between início e fim`) e classificando a rubrica pelo `evento.tipoevento` — **`1` = provento (o custo de remuneração)**, `3` = desconto (INSS/IRRF retido, vales, faltas), `líquido = provento − desconto`.

- **Excluir os tipos de folha que duplicam/não são pagamento** (`periodocalculo.codigotipocalc not in (8, 70, 71, 80)`): `8` adiantamento **antecipa** a mensal (somar os dois infla o provento — o adiantamento aparece na folha 8 e o salário cheio na 1), `70/71` provisão é accrual (e nem existe nesta base), `80` transferência é movimentação interna. Mantém mensal(1), 13º(20-23), férias(50/52), rescisão(60), intermitente(40), PLR(10/12), dissídio(6).
- **Encargos patronais (FGTS, INSS patronal, terceiros/RAT) NÃO são evento por funcionário** — não estão em `calculoevento`. O "custo" que sai daqui é a **remuneração** (proventos); o custo-empresa cheio precisa da apuração patronal (fonte ainda a mapear no banco — a doc fala em "Cálculo Patronal"/"débitos de INSS", tabela a achar). Não estime por alíquota (RAT/FAP/desoneração variam).
- **`evento`** (rubricas) é tratado como cadastro **global** (como cargo/função/escala). Classificar provento/desconto **por array de códigos** (`ce.codigoevento = any($prov)`) em vez de juntar `evento` evita o join numa varredura de 5M linhas — e evita o risco do join sem `codigoempresa` (o `provisoes.ts` junta `evento` só por `codigoevento`, mas como filtra tipo 70/71 inexistente, esse join nunca rodou de fato).
- **Setor/cargo/estab do custo**: juntar a view `funcionario` por `codigofunccontr` (mesma resolução do turnover). Dá o setor/cargo **atual**, não o da época da folha — aproximação aceitável para agregado gerencial.

Visto em: [[Navetech Hub]] (Folha → Custo de Folha).

## Quem lançou/calculou — produtividade do DP

Cada trabalho do DP mora numa tabela própria, e todas carregam a **auditoria embutida** (`codigousuario` → `usuario.nomeusuario`, `datahoralcto` = quando; ver [[Logs e auditoria no Questor]]). Isso é o que permite medir "o que cada pessoa do DP fez no período" sem tocar no `loggeral`:

- **Avisos prévios cadastrados** → `funcavisoprevio` (PK `(codigoempresa, codigofunccontr, seq, complementar)`): `codigocausa` → `causademissao`, `dataavprevio`, `dataresc`, e `enviouavisoprevioesocial` ('1'/'0').
- **Rescisões calculadas** → `rescisao`: `codigocausa`, `dataavprevio`, `dataresc`, `codigopercalculo`. `datahoralcto` é o carimbo do **último cálculo**.
- **Admissões feitas** → `funccontrato` (a própria linha do contrato): `dataadm`, `origemdado` (1 manual, 2 importado, 3 integração). Armadilha: `datahoralcto` é reescrito a cada edição do contrato, não só na criação — para "produtividade" (o que o DP mexeu no período) serve; para "admissões cujo fato ocorreu no período" filtre por `dataadm`.
- **Férias calculadas** → `reciboferias` (PK inclui `datainicial` = início do **período aquisitivo**): `datainicialferias`/`datafinalferias` (gozo), `datapgto`.

Recorte de produtividade = `datahoralcto::date between início e fim`, por `codigousuario`. Empresa via `codigoempresa` em cada tabela (não precisa da view). Usado no dashboard de Produtividade do DP em [[Navetech Hub]].

## Provisão de férias/13º NÃO é folha (tipo 70/71 não existe)

Armadilha cara: a lenda de `tipocalculo` diz "70/71 Provisão", mas **nesta base `periodocalculo` não tem UMA linha sequer de tipo 70/71** (verificado na base inteira, todas as empresas). Provisão de férias e 13º **não** é calculada como uma folha em `calculoevento` — mora em **tabelas próprias** por funcionário × competência:

- **`provisao13` / `provisao13rat`** — provisão de 13º (a `rat` é o rateio por centro de custo).
- **`provisaoferias` / `provisaoferrat`** — provisão de férias, **método clássico** (colunas `mes`, `mes1terc`, `mesinss`, `mesfgts`, `ajuste*`, `saldo`).
- **`provisaoferiasemdias` / `provisaoferiasemdiasrateio`** — provisão de férias, **método "em dias"** (91 colunas: `provisaomesremuneracao`, `provisaomestercoferias`, `provisaomesinss`, `provisaomesfgts`, `ajuste*`, `saldoanterior*`/`saldofinal*`, `pago*`, `diferencapagamento*`). Uma empresa usa **um** dos dois métodos — a SANTA ORANNA (1015) usa "em dias" e `provisaoferias` (clássico) veio **vazia** para ela (0 empresas usando clássico nesta base). Cubra os dois no código.

Chave: `(codigoempresa, codigofunccontr, compet)`, `compet` é **date = 1º dia do mês**. `provisaoferiasemdias` tem `codigoestab` direto; as outras não (filtre filial via `funccontrato`).

**Provisão do mês (accrual) = `provisaomes* + ajuste*`** (em dias) / `mes* + ajuste*` (clássico e 13), por componente: remuneração (+1/3), INSS, FGTS (PIS existe mas normalmente não tem conta contábil). Validado ao centavo contra o contábil de abr/2026 da 1015: férias 9762,86 + 13º 2928,72 = **12.691,58** = exatamente o lançado FP.

**Armadilha da conferência contábil × DP:** o movimento das **contas de provisão no contábil (origem `FP`)** mistura **accrual (provisão do mês) com realização (baixa quando férias/13º é pago)** — meses de pagamento dão movimento gigante ou negativo (ex.: 1015 fev/2026 contábil −42.666, DP accrual +6.940). Os dois lados só batem em mês **sem baixa** (abr/2026 bate porque só teve accrual). Pior: a **integração DP→contábil é mês a mês e pode estar dessincronizada** — jan-mar/2026 a 1015 tinha provisão calculada no DP e **zero** lançamento FP no contábil. Logo, conferir "provisão calculada × lançada" exige comparar **accrual com accrual** (isolar as baixas dos dois lados), não o movimento líquido cru da conta. Visto em: [[Navetech Hub]] (tela Provisões do Contábil).

## Rubricas — `evento`

Cadastro das rubricas/verbas (`codigoevento` → `descrevento`), com dezenas de flags de incidência (`inssmensal`, `fgtsmensal`, `irrfmensal`…). `tipoevento` classifica (observado): `1` provento/vencimento, `2` reembolso/salário-família, `3` desconto, `4` base de cálculo/informativo, `5` afastamento, `6` outros (banco de horas, abono). Confirmar por amostragem ao usar — os limites entre 3/4/6 são fluidos.

## View `funcionario` (atalho oficial)

A view `funcionario` junta `funccontrato` + `funcpessoa` + **o registro mais recente de cada tabela de vigência** (salário, cargo, local → `estab`, escala…) usando `max(datainicial)`. É a forma pronta de obter a "ficha atual" de cada funcionário sem reescrever essa lógica de vigência. Ótima referência de como as `func*` se ligam.

## eSocial (staging técnico — não é a folha)

`esocial*` (~429 tabelas) + `xml*` (~228) guardam **eventos, lotes e XML do eSocial** e seus retornos (`esocialret*`, `esocialdadoss*`, `esocialxml` ~10 GB de XML, `esocialeventoretornorubricas`…). É infraestrutura de transmissão ao governo. Para a folha "de negócio" (o que cada um ganhou), use `calculoevento` + `func*`, **não** as `esocial*`.

### Foi transmitido/aceito? A tabela de controle é `esocialtransacao`

Para saber o **status de transmissão** de um evento (S-2200 admissão, S-2299 rescisão, S-2206 alteração…), a fonte é `esocialtransacao` — uma linha por evento gerado: `(codigoempresa, codigoesocialtransacao)` PK, `evento` (`'S-2200'`…), `status` (smallint), `recibo`, `protocolo`, `codigorespostaesocial`, `datahoralcto`, e `track` = `'(FUNCCONTRATO::<n>)'`. **Regra prática: `recibo` preenchido = aceito pelo governo**; transação sem recibo = pendente/erro (status 13 + `codigorespostaesocial` 401 é rejeição); sem transação nenhuma = não enviado.

Ligar o evento ao contrato: pelas tabelas `esocialdadoss<NNNN>` (ex.: `esocialdadoss2200` para admissão) — `(codigoempresa, codigofunccontr) → codigoesocialtransacao` → `esocialtransacao`. Pegue a **última** por `datahoralcto` (retransmissões geram várias). Armadilha: **`xml2200evtadmissao` (e as `xml*` de layout) vêm VAZIAS** nesta base — são staging de montagem do XML, limpo após enviar; não sirva status delas. Verificado no dashboard de Produtividade do DP.

A **rescisão** liga pelo mesmo caminho no evento **S-2299** (tabela `esocialdadoss2299`, mesmo padrão do 2200 — a confirmar na base). Para um **panorama de conformidade** (sem ligar ao contrato): agrupe `esocialtransacao` por `evento` no período (`datahoralcto::date`) e classifique cada linha — `recibo` preenchido = **aceito**, sem recibo e `status = 13` = **rejeitado**, o resto = **pendente**. Dá o volume transmitido × resultado por tipo de evento. Visto em: [[Navetech Hub]] (Folha → eSocial).

## Contrato de experiência e o buraco de supervisor/e-mail

- **Experiência CLT = 45 + 45 dias**. O Questor guarda em `funccontrato.diaprorrogcontrexp` (observado `= 45` nas admissões recentes) e `justprorrog` (justificativa da prorrogação). Não há uma "data de fim de experiência" pronta — calcula-se `dataadm + 45` (1º período, decidir prorrogar) e `dataadm + 90` (efetivar ou desligar). `motivocontrat` e `codigotipocontr` classificam o contrato; empregado CLT filtra por `categoria = '01'`.
- **O Questor NÃO tem supervisor→funcionário nem e-mail de ninguém.** Procurado: só existe `codigosupervisorestagio` (para ESTAGIÁRIO, não CLT em geral); `organograma` não tem responsável; `funcpessoa` (96 colunas) **não tem coluna de e-mail**. Ou seja: qualquer automação de RH que precise "avisar o gestor de fulano" tem de **cadastrar gestor + e-mail por fora** (no banco do próprio app), mapeado ao setor (`organograma` por `codigoempresa, codigoestab, classiforgan`). O Questor dá o organograma e a lotação; a pessoa-gestora e o contato são conhecimento do app.
- Setor do funcionário na view `funcionario` = `(codigoempresa, codigoestab, classiforgan)` → `organograma.descrorgan`.
- **Armadilha `codigofuncao = 0` = "sem função".** A maioria dos contratos só tem cargo (`codigocargo`), não função — vêm com `codigofuncao = 0`. E existe um registro LIXO na tabela `funcao` com `codigofuncao = 0` (numa base, descrição "MOTORISTA DE CAMINHÃO MUNCK"); um `join funcao on codigofuncao = f.codigofuncao` casa esse lixo para TODO mundo sem função. Trate 0 como nulo: `join ... on fu.codigofuncao = nullif(f.codigofuncao, 0)`. Mesmo cuidado vale para qualquer código-FK que use 0 como "vazio".

## Por que importa

Base para relatórios de folha/RH (custo de pessoal, headcount, admissões/demissões, provisões) e integração com a contabilidade (origem `FP` em [[Módulo contábil do Questor]]). Modelo pessoa-vínculo-vigência é a chave para não errar nas consultas.

## Conexões
- Índice do banco: [[Banco Questor]] · Convenções: [[Panorama e convenções do banco Questor]]
- Integra em: [[Módulo contábil do Questor]] (origem FP)
- Padrão de vigência também citado em: [[Cadastros centrais do Questor - empresa, estab, pessoa]]
- Mapa: [[Banco Questor]]
