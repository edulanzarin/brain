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

Código em: `C:/Dev/navex` (sem remote ainda).

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
- Obrigações, Societário, Configurações e a administração (usuários, cargos,
  grupos) seguem no Nexo.

## Marca (24/09/2026)

- **Logo**: o monograma NX que o Eduardo desenhou, reconstruído em vetor a partir da geometria medida no PNG (retas a 45°, dois cantos arredondados no N, um no traço longo do X). O traçado automático saiu com ponta solta e haste quebrada; a geometria exata ficou fiel e limpa. Guia do navegador com o NX sobre quadrado branco; tom mais claro no tema noite.
- **Ícones dos setores** desenhados no idioma do logo, registrados por nome (`setor-contabil`...): razonete, %, pessoa, coração, calendário com visto, quadro societário, ajustes. [[Ícone de identidade é desenhado para o sistema, não puxado da biblioteca]].
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
- **Todos os módulos**: o banco do app roda em UTC e a hora sai 3 horas adiantada
  onde a consulta formata com `to_char` (auditoria, RH, produtividade do app)
  ([[Postgres de container nasce em UTC, e a hora formatada sem fuso mente]]).

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
  para fora da vista)
- [[Código sem cadastro se prova pelo comportamento do dado, não pelo rótulo herdado]]
- [[Falta de registro só prova algo dentro da janela em que a fonte era alimentada]]
  (o DP: férias e ativo)
- [[Filtro de tela não reusa o nome de parâmetro que o contexto já usa]]
- [[O número do painel sai da mesma conta da tela que ele abre]] (o RH e o DP)
- [[Quem escolhe de um cadastro lê, quem o administra escreve]]
- [[Postgres de container nasce em UTC, e a hora formatada sem fuso mente]]
- [[Parâmetro posicional não se renumera quando a coluna sai]]
- [[Aviso de alteração não salva no App Router intercepta o clique]]

## Próximos passos

- [ ] O Eduardo olhar o Contábil e dizer o que muda no visual.
- [ ] Administração (usuários, cargos, setores, grupos) e perfil.
- [ ] O Eduardo olhar o Fiscal.
- [ ] A moldura não tem modo celular: a barra lateral fica aberta e espreme a tela
  (vale para os dois módulos).
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
- [ ] Obrigações, Societário e Configurações.
- [ ] Remote no GitHub e deploy no servidor.

## Conexões
- Substitui: [[Navetech Hub]]
- Usa: [[Design]] · [[Infra]] · [[Banco Questor]]
- Mapa: [[Projetos]]
