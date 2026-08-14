---
tags: [tipo/atomica, camada/principio, dev/frontend]
criado: 2026-08-14
---

# A home de um módulo é o resumo que carrega sozinho; automação não abre sozinha

> Ao entrar num módulo, a primeira tela deve ser um **painel-resumo que carrega sozinho** — o retrato de "o que está acontecendo / o que cobra ação". Telas que são **automação** (precisam ser disparadas, gastam consulta pesada, produzem um efeito) NÃO viram a home e NÃO rodam sozinhas: o usuário escolhe rodá-las.

## A regra

Separe as telas de um módulo em dois tipos e trate a abertura de cada uma
diferente:

- **Leitura/panorama** — resume dados que já existem. Pode e deve **carregar
  automática** ao abrir. É a candidata a home.
- **Automação/ação** — dispara um processo (gerar arquivo, conciliar, importar,
  recalcular) ou depende de um recorte caro. **Não** abre sozinha: fica atrás de
  um "Executar"/filtro, e nunca é a primeira tela.

A home do módulo é sempre uma do primeiro tipo — um painel que junta as
**pendências** (o que precisa de ação, com atalho para a tela que resolve) e uns
**indicadores** (o que o módulo movimentou).

## Por que

Abrir um módulo tem de dar contexto imediato, não uma tela em branco esperando
um clique. Mas rodar automaticamente uma automação ao abrir é o oposto do que se
quer: gasta processamento à toa, pode ter efeito colateral, e rouba do usuário a
decisão de quando disparar. O critério é **"isto é ler ou é fazer?"** — ler abre
sozinho, fazer se dispara.

Isso também dá coerência entre módulos: cada um ganha o **mesmo** tipo de home
(um painel), e as automações vivem em seções irmãs. O painel não precisa dos
filtros globais — usa janelas próprias (mês corrente, pendências em aberto) e é
self-contained.

## Na prática

- Marque a seção-painel como **primeira** na ordem da sidebar (a home do módulo
  cai na 1ª seção visível) e como **sem filtro** (não espera "Executar").
- No painel, cada bloco é uma **consulta independente**: se um falha, os outros
  ainda aparecem — um painel não cai por um card.
- **Um atalho do painel leva à tela já com o filtro APLICADO**, não à tela
  pedindo "Executar". Carregue o link com o estado aplicado (ex.: `?ap=1` + o
  período/entidade que o card contou) — clicar cai direto nos dados. Um resumo
  que exige refiltrar ao clicar quebra a promessa de ser resumo.
- **A home pode ser PLURAL, recortada por papel**: um painel do colaborador (a
  fila de trabalho) e um do gestor (fila + indicadores/produtividade). São seções
  distintas, isoladas por permissão (o colaborador nem alcança o endpoint de
  gestão), e cada pessoa cai na que seu cargo libera — a mesma regra de "1ª seção
  visível". Não é um painel com `if (gestor)`: é [[Posse numa permissão binária é duas seções e recorte por linha]] aplicada à home.
- Exemplo do avesso: uma tela de **conciliação** precisa rodar (lê extrato, casa,
  gera arquivo) — não abre sozinha nem é home; o painel só mostra *quantas vezes
  rodou / quantos itens*, e linka para ela. Esse placar sai de graça da trilha de
  auditoria: [[A trilha de auditoria já é o placar de atividade, não crie tabela de métrica à parte]].

## Conexões
- Fecha pendência sem inferir: [[Uma pendência de prazo fecha por ato explícito, não por sinal inferido]]
- O placar do avesso: [[A trilha de auditoria já é o placar de atividade, não crie tabela de métrica à parte]]
- Visto em: [[Navetech Hub]] (Folha → Painel do DP; Contábil → Painel; Fiscal já tinha Painel)
- Mapa: [[Base]]
