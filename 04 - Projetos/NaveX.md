---
tags: [tipo/projeto, projeto/navex]
criado: 2026-09-24
status: ativo
codigo_em: C:/Dev/navex
---

# NaveX

> A plataforma de trabalho da Navecon sobre o banco do Questor, refeita do zero
> com cara nova. Substitui o [[Navetech Hub]] (o Nexo): mesma função, mesma
> camada de domínio, interface inteira nova.

Código em: `C:/Dev/navex`. Remote `git@github.com:edulanzarin/nexo.git`: desde
25/09/2026 o NaveX é o `main` do repositório do nexo, e o nexo2 ficou na tag
`nexo2-final` e no ramo `nexo2`.

## Por que existe

Em 24/09/2026 o Eduardo pediu para refazer "completamente TUDO com design mais
moderno", sem copiar a reescrita anterior do Nexo (a pasta `nexo`, que ele chama
de nexo1 e não aprovou), com todas as funções do nexo2 (o que está em produção).
Nome novo: NaveX. Ordem: Contábil primeiro.

## Estado atual

- Esqueleto, sistema de design, catálogo `/sistema` com prévia de tela, login,
  início e a moldura do módulo prontos.
- **Contábil completo (24/09/2026)**: as quinze seções do nexo2, 29 rotas,
  conferidas contra o Questor real (empresa 1200, ago/2026): conferência com
  1.918 notas, balancete fiscal, análise com o motor, pendências, auditoria,
  funcionários e as sete abas da Produtividade no escritório inteiro (2,24 mi de
  lançamentos). Tipos, lint, 126 testes e build limpos; nenhum erro de JavaScript
  nas páginas.
- Feito em paralelo por cinco agentes, um por grupo de seções, com um brief comum
  (paridade de função com o nexo2, interface só com os primitivos, sem build nem
  commit); a integração e as correções de primitivo ficaram numa passada só.
- **Fiscal completo (25/09/2026)**: as oito seções do nexo2 (Painel, Análises,
  Tributos, Conformidade, Notas Fiscais, Produtividade com sete abas e os dois
  Post Mortem), 40 rotas. Conferido contra o Questor real no escritório inteiro em
  ago/2026: 30 chamadas em 200, de 0,1 s a 15 s (a aba Impostos é a mais lenta),
  print de todas as telas nos dois temas e nenhum erro de JavaScript. Tipos, lint,
  126 testes e build limpos.
- Feito do mesmo jeito que o Contábil: a base (catálogo de seções, filtros do
  módulo, rotas) primeiro, depois três agentes em paralelo (visão e rotina,
  produtividade própria do Fiscal, produtividade espelhada do Contábil) e uma
  passada de integração com os prints.
- **DP completo (25/09/2026)**: as dez seções do nexo2 (os dois painéis,
  Rescisões a Pagar, Férias, eSocial, Rotatividade, Custo de Folha, Produtividade
  com seis abas e os dois Post Mortem), 21 rotas. Conferido contra o Questor real
  (empresa 1200 e escritório inteiro): todas as rotas em 200, de 0,1 s a 1,8 s,
  print de todas as telas, nenhum erro de JavaScript. Tipos, lint, 127 testes e
  build limpos. Mesmo método do Fiscal: base primeiro, três agentes em paralelo
  (painéis e rotina, rotatividade e custo, produtividade) e a integração.
- **RH completo (25/09/2026)**: as nove seções do nexo2 (Painel, Diretório,
  Experiência, Desempenho, Rotatividade, Formulários com três abas, Denúncias,
  Avaliações e Gestores), 24 rotas, e as quatro páginas abertas (formulário por
  link, denúncia, acompanhamento, avaliação de clima). O banco do app do NaveX
  não tinha nada do RH: a conferência semeou pelas próprias rotas (formulários,
  gestores, rodada de clima respondida, denúncias, rodada de desempenho, envio,
  regras automáticas), bateu nas rotas contra o Questor (todas em 200, até 0,2 s),
  tirou print de cada tela nos dois temas e no celular e esvaziou as tabelas no
  fim. Tipos, lint, 128 testes e build limpos. Mesmo método, com quatro agentes
  (pessoas, avaliações, formulários, canais).
- **Configurações completo (25/09/2026)**: a única seção do nexo2, Grupos de
  Empresa (lista, janela do grupo com o modo "todas, exceto", remoção), 3 rotas.
  Conferido contra o Questor real (1.587 empresas em 0,6 s): 26 checagens de API
  (nome repetido, grupo vazio, exceto resolvido, bloqueio pelo Post Mortem, 403
  para quem não tem a seção), Shift+clique e Enter exercidos pelo navegador, print
  nos dois temas e no celular, e o banco local vazio de novo no fim.
- **Societário completo (25/09/2026)**: no nexo2 o módulo é só o Post Mortem do
  setor (preencher e ler a equipe), e o domínio já tinha vindo com os outros
  setores, inclusive a nota de gravidade e quem avisou. O porte foi ligar páginas,
  rotas e seções. Conferido com três usuários de teste (analista e gestor do
  Societário, gestor do DP): 21 checagens de API (setor gravado, envio com o que
  falta, gravidade fora de 1 a 5, enviado que não se edita nem se apaga, o gestor
  do DP sem alcance nem pelo caminho do DP) e o envio feito pela tela com a nota
  clicada. A lista trazia um defeito de todos os setores, que a coluna a mais do
  Societário deixou visível: as colunas de texto não truncavam.
- **Obrigações e Administração completos (25/09/2026)**: com eles o NaveX tem
  tudo o que o nexo2 tinha, e a marca de módulo "pronto" e a faixa "Ainda no
  Nexo" saíram. Base escrita à mão (seções, rotas, dados), telas em quatro
  frentes paralelas, integração com print de cada tela nos dois temas.
  - **Obrigações**: a empresa da fila passou a vir do contexto do topo (no nexo2
    era um seletor da carteira do Acessórias dentro da tela), recortando por
    `codigoempresa` pelo mesmo funil de escopo das outras telas; a consulta ao
    vivo aceita qualquer CNPJ e preenche o da empresa do topo (a NAVECON tem três).
    A varredura do Acessórias ainda é do agendador do nexo2 (5h): dois agendadores
    dividiriam o limite da API. A competência vem em dia qualquer (há tarefa
    semanal), então fica a data inteira.
  - **Administração**: módulo `soAdmin` na mesma moldura dos outros (busca, troca
    de módulo, menu), fora da matriz de permissões; uma `cargo_secao` forjada para
    ele não abre nada. Cargo é página (a matriz passa de 49 seções), o resto é
    janela sobre a lista. "Grupos de Empresa" da administração virou "Grupos de
    Permissão", porque o nome batia com os grupos de negócio das Configurações.
    Toda escrita vai para a trilha, e ninguém consegue deixar o NaveX sem
    administrador ([[Invariante sobre o conjunto se confere no estado final, dentro da transação]]).
  - **Meu Perfil** no menu da pessoa: foto, senha (derruba as outras sessões) e
    sessões abertas. A foto nova aparece no menu na hora (a versão vai na URL).
  - Conferido: 78 checagens de API com cinco usuários de teste (fila por seção,
    empresa, grupo e escopo restrito; travas da administração; foto e perfil), uma
    consulta real ao Acessórias (NAVECON, 56 pendentes) e 20 telas no navegador,
    sem erro de JavaScript e com o banco local de volta ao retrato de antes.
- **Preparado para a troca (25/09/2026)**. O NaveX assumiu o repositório do nexo
  por um merge que liga as duas histórias e fica com a árvore do NaveX, então o
  `git pull` do servidor avança sem reset
  ([[Herdar um deploy é herdar o contrato dele, não só o domínio]]).
  - O agendador entrou no compose (`navex-scheduler`), atrás do perfil
    `agendador`, que só o `.env` do servidor liga. Na máquina de desenvolvimento
    ele não sobe, e o escritório nunca tem dois.
  - O roteiro da troca está no README. O schema é o mesmo, então a troca é dump do
    banco do nexo2 e restore no `navex-db` vazio, antes da primeira subida. Foi
    ensaiado contra um Postgres descartável: restore limpo, migrate sem nada a
    aplicar, contagens iguais.
  - No servidor o nexo2 roda com os nomes antigos (`questor-bi`, `questorbi`). Os
    comandos leem usuário e banco das variáveis do container, sem depender do nome.
- **Exportar em Excel e PDF (25/09/2026)**, pedido do time a partir do Diretório
  do RH. O `MenuExportar`, que já estava em 41 telas, ganhou os dois formatos sem
  que nenhuma tela mudasse, porque o tipo se reconhece nos valores
  ([[Excel e PDF saem da mesma tabela, e o tipo se reconhece no exportador]]).
  - Com vários recortes, o formato vira uma escolha no alto do menu, lembrada
    entre as telas.
  - A trilha registra o arquivo com a extensão.
  - Conferido baixando os três formatos do Diretório no navegador sem janela.
- **Complemento do extrato (25/09/2026)**, pedido do time da Conciliação. O
  Sicoob (e o Banco do Brasil, segundo eles) imprime o histórico abreviado numa
  linha e quem recebeu na de baixo; o leitor descartava a de baixo, e toda
  distribuição de lucro chegava igual. Agora é um campo próprio
  ([[Ler extrato bancário em PDF]]).
  - Aparece em segunda linha na tabela, no detalhe, na busca e na planilha, e vai
    junto no histórico do CSV do Questor.
  - A regra que só casa pelo complemento ganha da que casa pelo histórico
    ([[Especificidade se mede pelo que o campo distingue, não pelo tamanho do termo]]).
    A regra criada da linha nasce do favorecido.
  - O selo da folha passou a ler o complemento, que é onde está o nome.
  - Conferido com o PDF real da Tomaselli no container: 22 lançamentos, 11 com
    complemento, saldos fechando.
  - O Banco do Brasil ainda não tem leitor de PDF; falta um extrato de exemplo.
- **Data e hora iguais no servidor e no navegador (25/09/2026)**: toda tela com
  data e hora disparava erro de hidratação no container
  ([[Hora formatada no servidor sai no fuso do container e quebra a hidratação]]).
- **Módulo TI (25/09/2026)**, pedido do Eduardo com o cubo `ti.png` que ele fez.
  Primeira função: Equipamentos, em três abas (Inventário, Por Pessoa,
  Movimentações), e o histórico de com quem cada um esteve.
  - Com quem está é a última linha de `ti_movimentacao` (migration 040), nunca
    coluna do cadastro ([[Quando o passado importa, o estado atual é a última linha do histórico]]).
  - As especificações de cada tipo são dado em `ti-tipos`: tipo novo é uma
    linha, sem migration.
  - A pessoa vem do Diretório do RH por rota própria da TI, só nome, setor e
    cargo ([[Quem escolhe de um cadastro lê, quem o administra escreve]]).
  - Por Pessoa mostra quem está sem equipamento e quem saiu do Diretório com
    algo em mãos.
  - Conferido no container: 32 checagens de rota e tela, recusas incluídas, e o
    banco local de volta a zero.
  - **Quem não está no Diretório** (o terceirizado, o estagiário fora da folha),
    pergunta do Eduardo no mesmo dia: cadastro próprio da TI (migration 041),
    não texto livre nem PJ do RH. "Quem recebe" busca nos dois cadastros e
    cadastra alguém de fora dentro da entrega
    ([[Entidade auxiliar se cria no ponto de uso, não em tela própria]]).
    Encerrar o cadastro é o fim do vínculo, e o que ficou com a pessoa sobe
    como "A recolher", junto de quem saiu do Diretório.

## Marca (24/09/2026)

- **Logo**: o monograma NX que o Eduardo desenhou, reconstruído em vetor a partir da geometria medida no PNG (retas a 45°, dois cantos arredondados no N, um no traço longo do X). O traçado automático saiu com ponta solta e haste quebrada; a geometria exata ficou fiel e limpa. Na guia do navegador vai só o NX, sem fundo (o quadrado branco foi reprovado em 25/09), num azul mais claro quando o navegador está escuro.
- **Ícones dos módulos**: os cubos do nexo2 (sigla e cor por módulo), recortados e reduzidos a 192 px em `public/modulos`. Os ícones desenhados no idioma do logo (24/09) foram reprovados como genéricos em 25/09 e saíram. [[Ícone de identidade é desenhado para o sistema, não puxado da biblioteca]].
- **Títulos** com maiúscula nas palavras principais; mensagem de estado e rótulo de indicador seguem como frase.
- Roda no Docker desde a noite de 24/09 (`navex-app`, `navex-db`, `navex-migrate`); o banco portátil ficou desligado.

## Defeitos do nexo2 achados no porte (ainda abertos lá)

- Replicar plano e apagar regra de extrato não conferem o escopo de empresa; o
  aprendizado de CFOP e conta efetiva tem a corrida de apaga-e-insere.
- O editor do plano de contabilização apaga a fórmula de valor (`regraValor`) ao
  salvar um ajuste, e a divergência e o balancete fiscal dependem dela.
- `nota-itens` liberado só para a seção Notas: quem tem Conferência ou Pendências
  sem Notas recebe erro ao abrir os itens da nota.
- Editar regra de extrato pela linha mandava o histórico vazio e o apagava.
- A rota de lançamentos do balancete devolvia como total o tamanho da página
  (500), então a tela nunca sabia que a lista vinha cortada.
- **Conformidade do Fiscal lê `cdsituacao` errado**: chama o 5 de denegada e o 6
  de inutilizada e conta tudo que não é 0 como problema, então as notas
  complementares entram em "denegadas" (o número sai quase 4x maior) e as
  inutilizadas em "sem chave de acesso" (2.144 das 2.150 de 2026). Ver
  [[cdsituacao do Questor é o COD_SIT do SPED]].
- O filtro de espécie é ignorado sem aviso nas rotas que leem as tabelas de item e
  de imposto (impostos, produtos, CFOPs, tributos, devoluções, NCM): a tabela não
  tem espécie. O NaveX diz na tela quando isso acontece e trava o filtro em
  Tributos.

- **DP**: com um grupo de empresas no topo, a Produtividade e as Rescisões
  mostravam o escritório inteiro. Cada uma tinha uma cópia antiga do funil de
  escopo, sem grupo ([[Filtro transversal só é honesto se todo o funil o honra]]).
- **DP**: o filtro de estabelecimento da Rotatividade dava 400 em qualquer
  escolha, porque usava o mesmo nome de parâmetro da filial
  ([[Filtro de tela não reusa o nome de parâmetro que o contexto já usa]]).
- **DP**: férias vencidas contavam contrato sem folha havia meses e períodos
  anteriores à história do contrato no Questor: 1.469 no painel, 126 de verdade
  ([[Contrato sem demissão não prova funcionário ativo no Questor]]).
- **DP**: o eSocial contava o envelope de lote e a EFD-Reinf como evento
  pendente (39,6 mil no painel, 16 mil de verdade), a pendência obrigatória
  chamava o rejeitado de pendente, e o painel e a tela contavam status nulo
  diferente ([[esocialtransacao guarda lote e EFD-Reinf junto dos eventos do eSocial]]).

- **RH**: o painel contava experiência a decidir direto na tabela, então quem foi
  desligado no meio da experiência ficava pendente para sempre, e o atraso era o
  status gravado pelo job ([[O número do painel sai da mesma conta da tela que ele abre]]).
- **RH**: a decisão da experiência (efetivar, prorrogar, desligar) nunca chega à
  lista desde a migration 012: a resposta pelo formulário não grava a coluna
  `recomendacao`. O NaveX tira a decisão da pergunta marcada como decisão.
- **RH**: criar ou editar regra de envio automático falha no banco desde 28/08
  ([[Parâmetro posicional não se renumera quando a coluna sai]]).
- **RH**: quem tem só Desempenho, Formulários ou Avaliações recebe 403 nas listas
  de que escolhe, e as regras de envio automático não tinham seção dona
  ([[Quem escolhe de um cadastro lê, quem o administra escreve]]).
- **Configurações**: remover grupo usado num relatório do Post Mortem cai na
  página de erro (a chave estrangeira recusa e a action não trata); o nome único
  diferencia maiúscula, então "U FIT" e "U Fit" viram dois grupos no seletor; e o
  salvar faz um insert por empresa na transação (1.400 idas ao banco num grupo
  grande).
- **Obrigações**: sem responsável escolhido, a fila filtrava `resp_id = 0` e
  escondia as entregas com dono ([[Number de parâmetro ausente é 0, e 0 é um filtro válido]]);
  a retomada da varredura olhava a última parcial das 24 h mesmo depois de uma
  completa; e totais que falhavam voltavam zerados, com cara de fila em dia.
- **Administração**: excluir quem tem relatório do Post Mortem ou rescisão
  marcada dava erro de chave estrangeira; dava para ficar sem administrador
  (só a autoexclusão era barrada); excluir usuário, cargo, setor e grupo não
  pedia confirmação; a foto aceitava SVG, servido com o tipo que declarou; e
  mudança de permissão não deixava rastro na trilha.
- **Todos os módulos**: o banco do app roda em UTC e a hora sai 3 horas adiantada
  onde a consulta formata com `to_char` (auditoria, RH, produtividade do app)
  ([[Postgres de container nasce em UTC, e a hora formatada sem fuso mente]]).
- **Agendador**: o container roda em UTC, então, a menos que o `.env` do servidor
  defina `TZ`, os e-mails das 8h saem às 5h e a varredura do Acessórias das 5h
  sai às 2h ([[Agendador em container conta as horas em UTC]]). O padrão do
  endereço (`http://app:3000`) também é o nome do serviço do nexo2. No NaveX o
  compose fixa os dois.

## Infra

Slug `navex` · `navex-app` na **4083** · `navex-db` na **5083** · migrations no
serviço `navex-migrate`. Chassi em [[Infra]]. Roda no Docker desde 24/09/2026; o
banco fica publicado em 127.0.0.1:5083 pelo `docker-compose.dev.yml`. Um
`docker compose up --build navex-app` sem o override reinicia o banco sem a porta:
subir de novo com os dois arquivos para devolver a 5083.

## Decisões do Fiscal (25/09/2026)

- **Espécie da nota e valor/quantidade são filtros do módulo, na tela.** Só o
  Fiscal os lê, então não sobem para o contexto do topo; ficam no cabeçalho de cada
  tela e valem pelo módulo inteiro (NFS-e escolhida no Painel continua nas
  Análises e na Produtividade). Aplicam na hora, como todo filtro de tela.
- **Toda aba do Fiscal varre o escopo**: empresa e grupo no topo são filtro, nunca
  obrigação. Filial vale nas seções que leem nota; a Produtividade segue a do
  Contábil e não oferece filial.
- A seção `dados` aparece como **Notas Fiscais**, o mesmo explorador do Contábil;
  o id e o caminho continuam os do nexo2.
- O donut de espécie virou barra de composição com a lista embaixo; os quatro
  cartões e os dois resumos do Painel viraram uma faixa de indicadores, com o
  detalhe do movimento em modal.

## Início (25/09/2026)

O início deixou de listar as seções de cada módulo (o Eduardo: "aí não faz sentido
entrar lá"). Virou uma porta por módulo pronto, com onde a pessoa parou ou onde o
módulo abre, a busca Ctrl+K grande no alto e o Continuar ao lado. Os módulos que
seguem no Nexo ficam numa faixa compacta. Ver
[[A entrada leva ao módulo, não repete o que tem dentro dele]].

Na mesma leva a paleta (Ctrl+K) deixou de buscar empresa: empresa sozinha não é
destino, e trocar a da tela atual é do seletor do topo. Busca seções e, ao
digitar, as abas com caminho próprio; o que casa no nome vem antes do que casa
só na descrição.

## Decisões do DP (25/09/2026)

- **Três jeitos de ler o Questor no mesmo módulo**: os painéis carregam sozinhos;
  Rotatividade, Custo, Férias e eSocial são bancada de uma empresa; Rescisões e
  Produtividade são o escritório, com empresa ou grupo como filtro. Nenhuma seção
  oferece a filial do topo: a Rotatividade tem o estabelecimento no filtro dela.
- **Produtividade com uma aba por família de trabalho** (Movimentação, Férias,
  Folha, Cadastro, eSocial) e o trabalho escolhido dentro dela. As seis abas
  dividem a mesma execução (`execucaoCompartilhada` na aba), porque leem o mesmo
  resumo; a pessoa isolada vale nas seis.
- Os registros de cada trabalho (`dp-lista`), que o nexo2 tinha sem tela, viraram
  o detalhe em modal.
- As peças de pessoal (filtros, movimentações, quebras de turnover, ficha, drill)
  moram em `produto/pessoal` com `modulo: "folha" | "rh"`: o RH usa as mesmas.
- **O cron dos avisos de rescisão existe, mas o compose não sobe o agendador**:
  enquanto o nexo2 estiver no ar, é ele quem manda o e-mail; dois agendadores
  mandariam em dobro. As marcações de "paga" do nexo2 moram no banco dele, então
  a fila do NaveX mostra tudo como pendente até a troca de banco.

## Decisões do RH (25/09/2026)

- **A empresa do RH se escolhe na tela.** O dado é fixo nas três empresas da
  Navecon (NAVECON, FOUR, FINAVE), que o seletor do topo nem lista (ele mostra a
  carteira da sessão). As abas do RH não leem empresa do contexto; Diretório,
  Experiência e Rotatividade têm o seletor "Todas / NAVECON / FOUR / FINAVE"
  dentro da tela. É a única exceção à regra do contexto no topo.
- **Quase tudo carrega sozinho** (três empresas e um banco pequeno); só a
  Rotatividade lê o período do topo e pede Executar. A barra do topo passou a
  mostrar o período em aba que não lê empresa.
- **A Rotatividade é uma tela só para DP e RH** (`produto/pessoal/tela-rotatividade`),
  e o RH ganhou a faixa de filtros (setor, cargo, vínculo, horário), que não tinha.
- **Uma peça só desenha pergunta de formulário**: a prévia do editor, a página
  por link e a leitura de resposta usam a mesma, para a prévia não mentir.
- **Formulários em três abas** (Formulários, Envios, Automático), com o editor
  numa rota própria (`/rh/formularios/<id>`).
- Os crons de lembrete de experiência e de envio agendado existem, mas o compose
  não sobe o agendador enquanto o nexo2 estiver no ar.
- Sem SMTP configurado, o NaveX só registra o e-mail no log: dá para exercitar
  envio e lembrete sem ninguém receber nada.

## Decisões importantes

- **Domínio portado, interface nova.** `src/lib` (o SQL do Questor e os motores),
  as rotas de API e as 39 migrations vêm do nexo2 como estão. O schema é
  idêntico de propósito: no dia da troca, o NaveX assume o banco do app do
  nexo2 sem migrar dado. Os ids de seção também são os mesmos, porque são a
  chave de `cargo_secao`.
- **Três furos do nexo2 nascem fechados**: replicar plano de contabilização e
  apagar regra de extrato conferem o escopo de empresa, e o aprendizado de CFOP e
  de conta efetiva tem trava por empresa
  ([[Regravar o conjunto de uma chave com delete e insert exige trava por chave]]).
- **Contexto de trabalho no topo.** Empresa, grupo, filial e período moram na URL
  e valem para todas as seções do módulo; a tela não tem barra de filtro própria
  para eles. Cada aba declara no catálogo o que usa (período por dia, por mês ou
  nenhum; empresa obrigatória, opcional ou nenhuma; filial; verbo do botão).
- **Execução por botão na moldura.** A tela só monta depois da primeira execução
  (ou só com empresa, na bancada), e quando o recorte muda depois da execução o
  resultado continua na tela com o aviso de que mudou
  ([[Consulta pesada executa por botão, não por mudança de filtro]]).
- **Visual**: vidro sobre papel quadriculado que se apaga para baixo, dois temas
  (noite e dia), laranja da marca só na ação, azul para foco e seleção,
  Instrument Sans com eixo de largura (número de indicador a 82%), faixa de
  indicadores em vez de grade de cartões, paleta Ctrl+K com seções e empresas.
  A marca é um X de dois traços, laranja e azul, cruzando: a conferência entre
  duas fontes que o sistema faz o dia inteiro.

## Aprendizados (viraram notas)

- [[Escala própria do tema precisa ser ensinada ao tailwind-merge]]
- [[Fundo no body cobre a camada de z-index negativo]]
- [[Notação compacta do Intl muda com o ICU e quebra a hidratação]]
- [[Em tabela de layout automático o truncate só corta em coluna que aceita encolher]]
  (voltou no Fiscal: o ranking de pessoas e a carteira empurravam a última coluna
  para fora da vista; e de novo no Societário, onde a coluna de gravidade a mais
  empurrava o Atualizado da lista do post mortem)
- [[Código sem cadastro se prova pelo comportamento do dado, não pelo rótulo herdado]]
- [[Falta de registro só prova algo dentro da janela em que a fonte era alimentada]]
  (o DP: férias e ativo)
- [[Filtro de tela não reusa o nome de parâmetro que o contexto já usa]]
- [[O número do painel sai da mesma conta da tela que ele abre]] (o RH e o DP)
- [[Quem escolhe de um cadastro lê, quem o administra escreve]]
- [[Postgres de container nasce em UTC, e a hora formatada sem fuso mente]]
- [[Parâmetro posicional não se renumera quando a coluna sai]]
- [[Aviso de alteração não salva no App Router intercepta o clique]]
- [[Regex do matcher do Next em string precisa de barra dupla]] (o ícone da aba e as imagens
  de `public/` redirecionavam para o login desde o primeiro dia)
- [[No sharp o resize roda antes do extend, na ordem que for chamado]]
- [[Rascunho copia o dado do servidor uma vez, e a consulta por baixo não recarrega]]
  (a janela do grupo de empresa)
- [[Number de parâmetro ausente é 0, e 0 é um filtro válido]] (a fila do Obrigações)
- [[Invariante sobre o conjunto se confere no estado final, dentro da transação]]
  (sempre sobra um administrador)
- [[Lista marcável grande age sobre o que o filtro acha, não sobre o que a tela desenhou]]
  (voltou na matriz de permissões do cargo)
- [[Excel e PDF saem da mesma tabela, e o tipo se reconhece no exportador]]
- [[Agendador em container conta as horas em UTC]] (herdado do nexo2)
- [[No Git Bash, caminho Unix em argumento vira caminho do Windows]] (o ensaio da troca)
- [[Especificidade se mede pelo que o campo distingue, não pelo tamanho do termo]]
  (as regras da Conciliação com o complemento)
- [[Hora formatada no servidor sai no fuso do container e quebra a hidratação]]
- [[Quando o passado importa, o estado atual é a última linha do histórico]] (TI)

## Próximos passos

- [ ] O Eduardo olhar o Contábil e dizer o que muda no visual.
- [ ] O Eduardo olhar o Fiscal.
- [ ] A moldura não tem modo celular: a barra lateral fica aberta e espreme a tela
  (vale para todos os módulos; os prints de celular só valem para janela e
  página aberta).
- [ ] Corrigir a Conformidade no nexo2 enquanto ele for o que está no ar.
- [ ] O Eduardo olhar o DP.
- [ ] Levar ao nexo2 as correções do DP (grupo, estabelecimento, férias, eSocial)
  enquanto ele estiver no ar.
- [ ] O eSocial ainda conta cada retransmissão: falta achar a chave de evento dos
  periódicos (o `track` é da agenda, não do evento).
- [ ] O Eduardo olhar o RH.
- [ ] Levar ao nexo2 as correções do RH que quebram hoje: regra automática
  (placeholders), decisão da experiência em branco, painel de experiência, fuso.
- [ ] Os gestores ainda não se desativam (a API só apaga), e envio agendado não se
  cancela: nos dois o nexo2 também não tem a rota.
- [ ] O Eduardo olhar o Obrigações e a Administração.
- [ ] Levar ao nexo2 a correção da fila do Obrigações (`respId` ausente virando 0):
  é uma linha, e hoje a fila de lá esconde as entregas com dono.
- [ ] A troca no servidor (192.168.5.68): dump do nexo2 antes do `git pull`,
  restore no `navex-db`, `COMPOSE_PROFILES=agendador` no `.env` (liga o agendador
  do NaveX no mesmo gesto que desliga o do nexo2). Roteiro no README.
- [ ] Domínio `navex.navecon.net.br` pelo túnel do ts05: pedir ao TI o CNAME
  ([[O túnel publica alcançando o container pelo nome, sem abrir porta]]).
- [ ] O Eduardo olhar Configurações.
- [ ] O Eduardo olhar o Societário.
- [ ] Atualizar o Next (16.2.10 tem alerta crítico de desvio do proxy; a correção
  sai na 16.3.6, sem quebra de versão).
- [ ] Leitor do extrato do Banco do Brasil em PDF, com o complemento: pedir um
  extrato de exemplo (e o OFX do mesmo mês, para conferir).
- [ ] Fechamento: o amarelo acende com o estorno do saldo negativo de 01/09. A
  equipe pediu "débito e crédito no grupo 6"; o Eduardo vai confirmar o que é o
  grupo 6 (medidas em [[Fechamento mensal no Questor - a conta de Encerramento do Exercício]]).
- [ ] TI: o Eduardo olhar os Equipamentos. Ideias que ficaram de fora da
  primeira versão: importar a planilha de inventário que já existir e o termo
  de responsabilidade para imprimir na entrega.

## Conexões
- Substitui: [[Navetech Hub]]
- Usa: [[Design]] · [[Infra]] · [[Banco Questor]]
- Mapa: [[Projetos]]
