---
tags: [tipo/projeto, projeto/navecon-crm]
criado: 2026-08-06
status: ativo
codigo_em: ~/Dev/navecon-crm
---

# Navecon CRM

> CRM de atendimento da Navecon Contabilidade: o escritório atende as empresas-cliente
> por WhatsApp, organizado em departamentos (Contábil, Fiscal, Dep. Pessoal). Cinco telas
> — Início, Mensagens, Empresas, Tarefas, Relatórios — sobre Next.js e Postgres.

Código em: `~/Dev/navecon-crm`

## Estado atual

Esqueleto completo rodando local: build de produção passa, as seis telas (Início,
Mensagens, Empresas, Contatos, Tarefas, Relatórios) leem do Postgres e as ações de
atendimento (puxar, transferir, encerrar, reabrir, enviar mensagem/nota) mutam o banco
por Server Actions. Seed porta os dados do protótipo em `docs/`. Sem auth ainda — a
persona logada ("Marina Alves") é fixa. Sem remote git.

## Infra

Slug `navecon-crm` · app `navecon-crm-app` na `4046` · banco `navecon-crm-db` na `5046`.
Compose com app, db e migrate; imagem standalone. Chassi e mapa de portas em [[Infra]].

## Stack

Next.js 15 (App Router, Server Components, Server Actions) · React 19 · TypeScript ·
Postgres 17 em **SQL puro** via `pg`, sem ORM · Tailwind v4 (tokens em `@theme`, classes
em `@layer`).

## Decisões importantes

- **Estética recalibrada para densidade.** O protótipo (dialeto "Aurora": vidro, raios
  grandes, fontes grandes) era arejado demais para um CRM, que é tela cheia de informação.
  Mantive a identidade (roxo, cores de departamento, tokens claro/escuro) mas troquei
  vidro por superfície sólida, raio 20px→11px, base tipográfica para 13px, KPI 30px→22px.
  É exatamente o que a base já prega: [[Estética é por projeto, princípio de design é que se reusa]].
- **Navegação em sidebar, não barra no topo.** A pedido do Eduardo, com muita coisa
  chegando ao sistema. Grupos em acordeão (o da rota aberto) que colapsam para rail de
  ícones — ganha tela num CRM denso. Um catálogo de navegação num array só. Padrão em
  [[Sidebar em acordeão e layout de módulo]].
- **Contato é objeto de primeira classe, não campo da empresa.** Pertence a uma empresa,
  mas tem ficha própria (dados, CPF, aniversário, preferência de canal, atendimentos dele)
  e aba própria — os dois se linkam nos dois sentidos. Empresa também engrossou: nome
  fantasia, IE/IM, situação cadastral, endereço, contato, honorário/vencimento e validade
  do certificado. Migration aditiva (002), sem quebrar o que já existia.
- **SQL puro, sem ORM.** Migrator é um script de ~30 linhas; seed é node com `pg`. Ver
  [[Runner de migration em SQL puro dispensa o CLI do ORM]].
- **Histórico da ficha vem de `atendimentos`, não de tabela à parte.** Os atendimentos
  encerrados sem conversa alimentam o "Histórico" da empresa; a tela de Mensagens só
  mostra os que têm mensagem (`EXISTS`). Uma fonte da verdade, dois recortes.
- **Seed com timestamps relativos a `now()`** — a fila mostra "espera 2h", o chat agrupa
  por dia, sem congelar numa data fixa. A UI parece viva a cada visita.
- **Agregados de relatório em tabelas próprias** (`volume_diario`, `pico_horario`,
  `dept_metricas`), separados das telas operacionais que são calculadas ao vivo. As médias
  (1ª resposta, resolução) são ponderadas por volume, computadas de verdade, não fixas.

## Aprendizados (viraram notas)

Só links. O texto mora na nota atômica.

- [[Uma faixa de portas por projeto]] — ganhou o caso das portas *unsafe* do navegador:
  4045 é recusada (o Next nem sobe), por isso o par ficou 4046/5046.

## Próximos passos

- [ ] Auth real (hoje a persona é fixa em `lib/session.ts`)
- [ ] Integração real com o WhatsApp (hoje as mensagens são seed)
- [ ] Busca funcional (empresa/CNPJ/contato) — os campos de busca são decorativos
- [ ] CRUD de empresa/contato/tarefa (botões "Nova …" ainda sem ação)
- [ ] Remote git + push

## Conexões
- Usa: [[Design]] · [[Infra]]
- Base: [[Configuração vem do ambiente, não do código]] · [[Estética é por projeto, princípio de design é que se reusa]]
- Técnicas: [[Sistema de cores e tema do dashboard]] · [[Sidebar em acordeão e layout de módulo]] · [[Runner de migration em SQL puro dispensa o CLI do ORM]] · [[Next.js standalone no Docker e o outputFileTracingRoot]]
- Mapa: [[Projetos]]
