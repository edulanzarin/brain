# Brain — Segundo Cérebro do Eduardo

Vault Obsidian em `~/Dev/brain`. É a base de conhecimento pessoal do Eduardo (dev).
Quando você (Claude) trabalhar em QUALQUER projeto dele, parte do trabalho é **capturar
o conhecimento aqui**, seguindo estas regras.

O objetivo não é acumular notas. É que daqui a um ano exista uma **base** de onde
qualquer sistema novo possa ser construído sem redescobrir nada.

---

## 1. A regra que manda em todas as outras: primeiro a base

**Ao aprender algo, a pergunta nunca é "em que projeto guardo isso?". É "em que camada
isso vive?".**

Sempre puxe o aprendizado pro nível **mais primitivo em que ele ainda seja verdade**:

| A lição continua valendo... | Camada | Onde |
|---|---|---|
| trocando o framework e a linguagem | **princípio** | `01 - Base/` |
| só citando a ferramenta | **técnica** | `02 - Técnicas/` |
| só naquele sistema | fica na nota do projeto, não vira atômica | `04 - Projetos/` |

Isso vale **no meio da construção, não só no fim**: se aparece um conceito reutilizável
enquanto o sistema é escrito, registre na base **na hora** e faça o projeto só linkar.
Deixar pra depois é como o conhecimento acaba preso no projeto.

Prioridades ao escrever: **primitivo > reutilizável > organizado > robusto > escalável**.
Uma nota a mais na base vale mais que dez notas soltas.

---

## 2. Estrutura

```
00 - Entrada/           captura rápida; processar e esvaziar
01 - Base/              PRINCÍPIOS — o tronco
    Cérebro/            como este vault funciona
    Design/             como a tela se organiza
    Infraestrutura/     como um projeto sobe
    Segurança/
    Ofício/             como eu trabalho
02 - Técnicas/          PADRÕES — aplicação concreta
    Design/
        Cor e tema/
        Layout e espaço/
        Componentes/
        Estados e feedback/
    Frontend/           React, Next, TypeScript
    Backend/            Node, rotas, arquivos, integrações
    Banco de dados/     Postgres, query, modelagem
    Infra e deploy/     Docker, Compose, migrations
03 - Referência/        schemas de sistemas externos
    Banco Questor/
04 - Projetos/          uma nota por sistema ativo
05 - Arquivo/           projetos encerrados
06 - Mapas/             MOCs
07 - Pensamentos/       como o Eduardo pensa
Templates/
```

**Uma pasta nova nasce quando um assunto passa de ~6 notas**, não antes. Pasta com duas
notas é organização teatral.

---

## 3. Direção do link

O link **sobe**: projeto → técnica → princípio. É obrigatório — é o que dá estrutura ao
grafo. O link **desce** é opcional e enxuto: um princípio cita uma ou duas técnicas de
exemplo, nunca todas.

### Regras duras

- **Nota de conhecimento não leva tag `#projeto/*`.** Só a nota do projeto leva.
  (Exceção: `#banco/questor`, que marca domínio externo, não projeto.)
- **Nota de conhecimento não tem "Faz parte de: [[Projeto]]" nem "Origem: [[Projeto]]".**
  Uma técnica de Postgres não faz parte do Questor BI. Use **"Visto em:"** — o projeto é
  evidência de onde apareceu, não dono. O backlink do Obsidian já mostra a origem, e
  mostra *todos* os projetos que usaram, não só o primeiro.
- **Projeto não linka projeto.** Dois sistemas que não trocam dado não se linkam, mesmo
  compartilhando stack, visual ou servidor. O que têm em comum não é um o outro — é a
  base. Errado: `Navedesk — Reusa o visual de: [[Navetech Hub]]`. Certo:
  `Navedesk — Usa: [[Design]]`. (O exemplo cita `[[Navetech Hub]]`, antigo "Questor
  BI".) Só linke projeto a projeto se houver relação real:
  consome a API do outro, compartilha banco, ou substituiu o outro.
- **Projeto arquivado não leva conhecimento junto.** Se levar, o link estava errado.

Notas-mestras: `[[Camadas do conhecimento - princípio, padrão, aplicação]]` ·
`[[Conhecimento pertence à base, não ao projeto]]`

---

## 4. Quando promover à Base

Princípio novo entra quando o **mesmo aprendizado aparece pela segunda vez**, em
contexto diferente. Antes disso é técnica. Princípio sem dois casos concretos costuma
ser regra inventada, não aprendida — e regra inventada é o que faz base virar entulho.

Toda técnica deve ter `- Princípio: [[...]]`. Se nenhum princípio cobre, uma de duas:
falta um princípio (escreva) ou é folha isolada (marque como tal). **Nunca invente um
link falso só pra preencher o campo** — link falso é pior que campo vazio.

---

## 5. Nota que mistura assunto se ramifica

Se uma nota cobre duas funções (layout **e** controles **e** dados), quebre em irmãs e
transforme a original em **índice**, mantendo o título antigo pra não quebrar links.
Foi o que se fez com `[[Padrões de componentes de dashboard]]`.

---

## 6. Convenções de nota

```
---
tags: [tipo/atomica, camada/principio, design]
criado: AAAA-MM-DD
---
```

- `tipo/*`: `atomica`, `projeto`, `moc`, `pensamento`
- `camada/*`: `principio`, `padrao`, `referencia`
- Tema livre: `design`, `infra`, `dev/frontend`, `dev/backend`, `armadilha`, `sql`
- **Não usar a tag `conceito`** — quase tudo é conceito, então ela não separa nada.

Bloco final padronizado, sempre `## Conexões` (nunca `## Links`), nesta ordem:

```
## Conexões
- Princípio: [[ ]]      obrigatório em técnica
- Depende de: [[ ]]
- Irmã: [[ ]]
- Visto em: [[ ]]       projeto onde apareceu — evidência, não dono
- Mapa: [[ ]]           obrigatório em toda nota
```

Templates em `Templates/`: `Princípio`, `Técnica`, `Projeto`. Cada um traz as
regras num comentário HTML no fim — ler antes de salvar.

---

## 7. Princípios de escrita

1. **Uma ideia por nota.** Duas ideias, duas notas, ligadas.
2. **Toda nota tem entrada e saída.** Nota órfã é conhecimento perdido.
3. **Palavras próprias**, não copiar/colar. Resumir, explicar, conectar.
4. **Título afirmativo e específico** — "Porta interna é constante, porta externa é
   configuração", não "Notas de Docker". O título é o texto do link.
5. **Português**, sem emoji em título nem no corpo.
6. **Qualidade > volume.** Antes de criar, buscar se já existe — e atualizar em vez de
   duplicar.

---

## 8. Convenções de projeto (consultar antes de criar sistema novo)

Já decididas, documentadas em `[[Infra]]`. Seguir sem perguntar:

- **Slug kebab-case** governa tudo: `prospects` → `prospects-app`, `prospects-db`,
  `prospects-migrate`, rede `prospects-net`. `container_name` explícito e
  `COMPOSE_PROJECT_NAME` no `.env`.
- **Porta interna constante** (3000 app, 5432 banco); **externa é configuração**:
  `ports: ["${APP_PORT:-40xx}:3000"]`.
- **Um script `dev` só**: `"dev": "next dev -p ${PORT:-40xx}"`. Outra porta é
  `PORT=3000 npm run dev`. Nunca criar `dev:3000`, `dev:3001`.
- **Faixa reservada por projeto**: app em `4xxx` (4001–4999), banco espelha trocando o
  `4` inicial por `5` (app `4004` → banco `5004`), e um terceiro serviço que precise de
  porta (agendador, worker, fila) vai pra `6xxx` com os mesmos três dígitos (`6004`). A
  faixa é o papel, não a ordem: segundo processo web continua em `4xxx`. Antes de
  reservar `6xxx`, confirme que o serviço precisa mesmo de porta publicada — agendador
  normalmente não precisa. Consultar e atualizar `[[Uma faixa de portas por projeto]]`.
- Dentro da rede do compose, serviços falam por **nome + porta interna**
  (`prospects-db:5432`), nunca pela porta publicada.

---

## 9. Cores do grafo

Grafo nativo (`.obsidian/graph.json` → `colorGroups`): **só os projetos têm cor, o
resto é cinza**. Cada projeto é um ponto colorido; todo o conhecimento (princípio,
padrão, referência, pensamento, moc) fica no cinza padrão, sem grupo de cor. Os
projetos saltam, o tronco de conhecimento é fundo neutro.

- Um projeto, uma cor, **só na nota do projeto**: navetech-hub azul `3900150` ·
  navedesk rosa `16007790` · cofre-digital verde-limão `8378422` ·
  navecon-controller coral `14707802`
- **Projeto novo**: adicionar um `colorGroup` com `"query": "tag:#projeto/<slug>"` e
  cor nova distinta. Nada de grupo de cor para camada ou tipo.
- Templates e `CLAUDE.md` ficam fora do grafo pelo filtro `search`.

Não mexer em `.obsidian/` sem pedido — exceto `graph.json` `colorGroups`, já autorizado.

---

## 10. Fluxo ao concluir trabalho num projeto

1. **Aprendizados vão pra camada certa primeiro** (base, se couber).
2. **Nota de projeto** em `04 - Projetos/` — estado, decisões, próximos passos e links
   pros aprendizados. O texto do aprendizado mora na nota atômica, não aqui.
3. **Linkar** projeto → notas → mapas, e atualizar o mapa correspondente em `06 - Mapas/`.

---

## 11. Versionamento

Remote SSH: `git@github.com:edulanzarin/brain.git` (chave já cadastrada no GitHub).

- **Sempre commitar E dar push** ao terminar de escrever/atualizar notas.
- Identidade: `edulanzarin <edulanzarin@outlook.com>`.
- Trabalho grande (reorganização ampla) vai em **branch**, não direto na `main`.
