---
tags: [tipo/projeto, projeto/navedesk]
criado: 2026-07-20
status: ativo
codigo_em: ~/Dev/navedesk
---

# Navedesk

> Sistema de chamados (help desk) interno da Navecon. Colaboradores abrem
> chamado para o TI, acompanham o andamento e recebem notificação a cada
> movimentação; o TI atende, prioriza, classifica e mede o atendimento.
> Construído do zero em jul/2026.

Código em: `~/Dev/navedesk`

## Estado atual

Funcionando ponta a ponta em Docker, acessível na rede em `http://<ip>:4001`.
Fluxo completo validado com navegador de verdade: login, abertura com anexo,
comentário, nota interna, mudança de status, avaliação, notificação entre dois
usuários e isolamento de permissão.

## O que tem

- **Chamados** com número sequencial, prioridade (Baixa/Média/Alta/Urgente),
  categoria, anexos (imagem com miniatura + lightbox, PDF, documentos) e
  **prazo de SLA** calculado pela prioridade — quem passa aparece como
  *Atrasado*.
- **Ciclo de vida** `Aberto → Em andamento → Aguardando → Resolvido → Fechado`
  (+ `Cancelado`), com transições validadas no servidor.
- **Comentários** na linha do tempo, com **nota interna** visível só para o TI
  (inclusive os anexos dela).
- **Histórico**: toda mudança vira evento (quem, o quê, quando).
- **Notificações in-app**: sino com contador, polling de 30s só com a aba
  visível. Nada de e-mail — decisão dele, para não depender de SMTP.
- **Avaliação** 1–5 do solicitante ao resolver; a média alimenta o painel.
- **Painel** (TI/gestor): fila, atrasados, resolvidos em 30d, tempo médio,
  distribuição por status/prioridade, satisfação.
- **Administração**: usuários, setores, categorias (com SLA próprio) e
  configuração (SLA por prioridade, domínio de auto-cadastro, limites de anexo,
  fechamento automático).
- **Rotinas horárias** dentro do próprio processo Next: fechamento automático,
  aviso de SLA estourado, limpeza de sessão. Sem cron, sem worker.

## Decisões importantes

- **Três perfis**: TI (`ADMIN`), Gestor (vê o próprio setor) e Usuário. O
  Eduardo escolheu já deixar o Gestor pronto mesmo sendo o único do TI hoje —
  custa quase nada e evita refazer depois.
- **Auto-cadastro restrito ao domínio** `@navecon.net.br`, em vez de o TI
  cadastrar um a um. O TI é notificado de cada conta nova para poder moderar.
- **Sem framework de auth e sem biblioteca de UI.** Sessão própria (JWT `jose`
  em cookie `httpOnly`) e componentes seguindo o design system do
  [[Questor BI]]. Coerente com [[Manter o tooling enxuto e o conhecimento no cérebro]].
- **Sessão espelhada no banco**: desativar usuário derruba o acesso na hora, em
  vez de esperar o cookie vencer.
- **Permissões num arquivo só** (`src/lib/permissoes.ts`): a listagem usa o
  filtro Prisma e o detalhe usa a função — mesma fonte, sem regra duplicada.
  Chamado que o usuário não pode ver responde **404**, não 403.
- **Anexos em disco**, não no banco: volume Docker, nome no disco é UUID (o
  nome original nunca toca o filesystem) e a entrega passa por rota que confere
  permissão a cada request. Ver [[Servir anexo por rota com checagem de permissão]].
- **Leitura em Server Component, escrita em Server Action.** Só duas rotas de
  API, e porque precisam mesmo: polling do sino e download de anexo.

## Deploy

`docker compose up -d --build` é o único comando — inclusive em máquina nova.
Um container `migrate` aplica as migrations e roda o seed, e o app só sobe
depois dele terminar com sucesso. Ver
[[Migrations em container próprio no Docker Compose]].

Detalhes que valem lembrar:

- O **seed é idempotente** e nunca sobrescreve conta existente: mudar
  `ADMIN_SENHA` no `.env` depois do primeiro boot **não** troca a senha de quem
  já existe (de propósito — seria uma porta dos fundos).
- O Postgres não publica porta: só o app fala com ele pela rede interna do
  compose. Apenas a 4001 é exposta.
- `COOKIE_SEGURO=false` porque a rede é HTTP puro; com `true` o navegador
  descartaria o cookie e o login entraria em loop.
- A migration inicial foi gerada **offline** com `prisma migrate diff
  --from-empty`, sem precisar de banco no ar.

## Stack

Next.js 16 (App Router) · React 19 · TypeScript · Tailwind v4 · Prisma 6 ·
PostgreSQL 17 · Docker Compose. Ícones lucide, datas com date-fns, validação
com zod, senha com bcryptjs.

## Aprendizados (viraram notas atômicas)

- [[Migrations em container próprio no Docker Compose]]
- [[Classes de componente vão em @layer components no Tailwind]]
- [[React reseta o formulário ao fim de uma Server Action]]
- [[Componente de ícone não atravessa a fronteira server-client]]
- [[Servir anexo por rota com checagem de permissão]]

## Próximos passos possíveis

- Base de conhecimento / FAQ ligada às categorias (chamado recorrente vira
  artigo).
- Relatório por período exportável para o gestor.
- E-mail como segundo canal de notificação, se um dia fizer falta.

## Conexões

- Reusa o visual de: [[Questor BI]] · [[Design]]
- Mesma família de deploy: [[Navecon Controller]]
- Faz parte de: [[Desenvolvimento]]
