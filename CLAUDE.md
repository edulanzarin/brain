# Brain — Segundo Cérebro do Eduardo

Este vault é o **segundo cérebro** do Eduardo (dev). O objetivo é virar uma rede
crescente e interligada de conhecimento: tudo que aprendemos, decidimos e
construímos nos projetos vira nota aqui, e as notas se conectam entre si até
formar um grafo denso (a "rede neural").

Quando você (Claude) trabalhar em QUALQUER projeto/tarefa do Eduardo, parte do
trabalho é **capturar o conhecimento aqui**, seguindo as regras abaixo.

## Princípios

1. **Notas atômicas** — uma ideia por nota. Se uma nota fala de duas coisas,
   quebra em duas e liga com `[[link]]`.
2. **Linkar sem medo** — toda nota deve apontar para pelo menos 2 outras. É o
   linking que cria o grafo. Link para nota que ainda não existe é OK (vira
   nota-fantasma a preencher depois).
3. **Escrever com as próprias palavras** — não copiar/colar cru. Resumir,
   explicar, conectar.
4. **Título é a interface** — títulos claros e específicos ("Debounce vs
   Throttle", não "Notas JS"). O título é como o link vai ser referenciado.
5. **Português** por padrão nos títulos e no corpo.

## Estrutura (PARA + Zettelkasten)

- `00 - Inbox/` — captura rápida, bagunça temporária. Processar e mover depois.
- `01 - Projetos/` — projetos com objetivo e prazo (uma pasta ou nota por projeto).
- `02 - Áreas/` — responsabilidades contínuas sem fim (ex: Carreira, Saúde, Finanças).
- `03 - Recursos/` — conhecimento atemporal e reutilizável (as notas atômicas / zettels).
  - `03 - Recursos/Banco Questor/` — referência do schema do banco Questor (conexão, modelo fiscal, impostos, canceladas/devoluções, receitas SQL). Todo aprendizado novo sobre o banco vai/atualiza aqui.
- `04 - Arquivo/` — projetos concluídos ou coisas inativas.
- `05 - MOCs/` — Maps of Content: notas-índice que agrupam e dão navegação ao grafo.
- `Diárias/` — notas diárias (log do dia, pega no template Diária).
- `Templates/` — modelos de nota.

## Fluxo quando eu trabalho num projeto do Eduardo

Ao concluir um trabalho relevante (feature, bug difícil, decisão de arquitetura,
aprendizado novo), ANTES de encerrar:

1. **Nota de projeto** em `01 - Projetos/` — o que é, estado atual, decisões,
   próximos passos. Usa o template `[[Projeto]]`.
2. **Notas atômicas** em `03 - Recursos/` para cada conceito/aprendizado
   reutilizável que apareceu (ex: um comando, um padrão, uma armadilha).
3. **Linkar** a nota de projeto ↔ as notas atômicas ↔ MOCs relevantes.
4. **Atualizar o MOC** correspondente em `05 - MOCs/` com um link pra nota nova.
5. Se for algo grande, registrar uma linha na diária de hoje em `Diárias/`.

Regra de ouro: **nada entra no vault sem pelo menos um link de entrada e um de
saída.** Nota órfã é conhecimento perdido.

## Convenções de nota

Frontmatter mínimo:

```
---
tags: [tipo/atomica]   # ou tipo/projeto, tipo/moc, tipo/diaria, tipo/pensamento
criado: AAAA-MM-DD
---
```

Tags por tipo: `tipo/atomica`, `tipo/projeto`, `tipo/moc`, `tipo/diaria`,
`tipo/pensamento`. Tags por tema livres: `#dev/frontend`, `#dev/backend`,
`#conceito`, etc.

### Pensamentos (como o Eduardo pensa)

Além de projetos e conhecimento técnico, o cérebro guarda **pensamentos** do
Eduardo: opiniões, princípios, preferências de trabalho, decisões de gosto que
ele expressa nas conversas. Regras:

- Vão em `03 - Recursos/` com tag `tipo/pensamento`, SEMPRE separados de projeto.
- Uma ideia por nota, com as palavras dele quando der. Título afirmando o
  pensamento (ex: "Prefiro ferramenta enxuta a config poluída").
- Linkar ao que motivou (um projeto, um conceito) — nunca órfão.
- Separar pensamento de coisa aleatória: se não tem valor reutilizável, não vira
  nota (ou fica no `00 - Inbox/` até virar algo). Na dúvida, oferecer ao Eduardo.

## Cores do grafo

O grafo é colorido **por projeto** (grafo nativo do Obsidian, em
`.obsidian/graph.json` → `colorGroups`). Objetivo: cada projeto é um cluster de
UMA cor só; a estrutura (MOCs) fica cinza; nada de arco-íris.

- **Cada nota de um projeto** leva a tag `#projeto/<slug>` (ex: `#projeto/questor-bi`),
  inclusive as atômicas — é isso que pinta o cluster inteiro de uma cor.
- Uma atômica reutilizada por vários projetos pode ter várias tags `#projeto/*`.
- **Ao criar projeto novo**: adicionar um `colorGroup` em `graph.json` com
  `"query": "tag:#projeto/<slug>"` e a próxima cor curada da paleta abaixo, e
  taggar as notas dele. Paleta (rgb decimal): azul `3900150`, teal `1352385`,
  âmbar `16090144`, rosa `16007790`, ciano `399329`, verde-limão `8378422`.
- Cinza dos MOCs: `7041664`. Roxo de pensamento: `11032055`.
- Templates e `CLAUDE.md` ficam ocultos do grafo pelo filtro `search`.
- O plugin Extended Graph está instalado mas com coloração desligada — as cores
  vêm do grafo nativo (mais confiável). Não reativar sem motivo.

## O que NÃO fazer

- Não criar dezenas de notas vazias de uma vez. Qualidade > volume.
- Não duplicar: antes de criar, checar se já existe nota do assunto (buscar).
- Não deixar nota sem link.
- Não mexer em `.obsidian/` sem o Eduardo pedir — EXCETO o esquema de cores do
  grafo (`graph.json` `colorGroups`), que ele já autorizou manter.

## Versionamento (git)

Este vault é um repositório git com remote `https://github.com/edulanzarin/brain.git`.

- **Sempre commitar E dar push** ao terminar de escrever/atualizar notas — o Brain
  fica sincronizado no GitHub.
- Identidade: `edulanzarin <edulanzarin@outlook.com>`.
- `.gitignore` cobre estado de UI do Obsidian (`workspace.json`) e binários
  grandes de plugins/temas (re-baixáveis) — versiona notas + config essencial
  (incl. `graph.json`).
