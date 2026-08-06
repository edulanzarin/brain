---
tags: [tipo/projeto, projeto/navehub]
criado: 2026-08-03
status: ativo
codigo_em: ~/Dev/navehub
---

# Navehub

> CRM + atendimento por WhatsApp para escritório de contabilidade, multi-tenant
> desde o dia 1 (feito pra vender a outros escritórios). Atendimento e CRM num
> produto só. **Substitui o [[navetalks]]** (recomeço deliberado, ago/2026): o
> navetalks era só atendimento; o Navehub une atendimento + CRM. A relação é
> real (um substituiu o outro), por isso o link — a regra "projeto não linka
> projeto" abre exatamente para este caso.

Código em: `~/Dev/navehub` · sem remote ainda (commit local em dia).

## Estado atual

**Refeito do zero (ago/2026)** — o Eduardo pediu rebuild com os melhores métodos,
escalabilidade, design moderno e **muitos modais**; a fundação anterior tinha sumido
(pasta vazia). Estado atual de pé e verificado: `docker compose up -d --build` sobe
migrate+app na 4030, build de produção e typecheck limpos, `/conversas` em 200 SSR
contra o Postgres seedado, e a **regra de fila conferida ao vivo** (Bruno, só Geral,
não vê o DP nem na lista nem abrindo a conversa por URL direta; Carla, Geral+DP, vê).

- **Inbox de fila compartilhada estilo Gestta** (2ª iteração, ago/2026 — o Eduardo
  achou a 1ª confusa, com filtros redundantes). Duas colunas (lista + conversa),
  ficha do contato em **modal**. A lista virou **abas**: **Fila de espera** (sem dono,
  com atividade), **Meus chats** (dono = eu) e **Equipe** (só supervisor/admin: o
  atendimento do time). Sumiram os filtros de estado/responsável e as pills; fila
  virou **select** (escala pra muitas). Envio **otimista**; SLA por conversa.
- **Posse explícita**: o que ninguém puxou fica na espera "apitando". Puxar (ou
  **enviar**) me torna dono e tira da espera. Fio contínuo por (fila, contato) — uma
  conversa persistente por par (UNIQUE); **"abrir conversa"** é find-or-create e traz
  o histórico (nunca "nova").
- **Permissão robusta escopada ao setor**: papel global só admin|funcionário; o
  **supervisor é por fila** (`fila_membro.supervisor`), não cargo global. Funcionário
  comum vê espera + as suas; **lê** conversa de colega dentro da fila (modo leitura,
  sem composer), mas não manda nem puxa. **Transferir** (pra qualquer usuário, de
  qualquer setor) é só do supervisor da fila ou admin. Tudo trancado no servidor; a
  UI recebe flags (`pode_responder`/`pode_gerenciar`/`sou_dono`). Verificado ao vivo:
  Bruno (Geral comum) não vê o DP nem a conversa da Carla na lista, mas lê a da Carla
  em modo leitura; Diego (supervisor Geral) vê a aba Equipe do Geral e transfere.
- **Modais**: ficha do contato (notas via TanStack, edita, timeline), transferir
  (todos os usuários da org, busca), abrir conversa (picker), respostas salvas,
  confirmação do finalizar.
- **Multi-tenant por `org`**: toda linha carrega `org_id`.
- Envio ao WhatsApp é um **seam** (`deliverText`), ainda sem conector (fica `pendente`).
- Login ainda não existe: em dev o "usuário atual" vem de um cookie (seletor "Ver
  como") pra demonstrar posse e permissão ao vivo.
- **Contato é da org, não da fila** (correção ago/2026): o número (fila) é só o canal.
  Módulo **Contatos** (menu próprio) é o diretório de clientes com a ficha — dados,
  **conversas do cliente** (em vários números) e anotações. **"Abrir conversa" saiu
  da inbox** e virou ação da ficha: escolhe o número e abre/reusa o fio
  (find-or-create). A inbox ficou só pra atender o que já existe.
- **Home = Painel operacional** (ago/2026): a `/` deixou de redirecionar e virou
  painel escopado por fila — KPIs (espera, SLA, meus, ativas), "espera por fila",
  "carga da equipe" (só supervisor) e a lista acionável "precisam de atenção".
  Gráficos em SVG/CSS com tom único (cor de fila não separa sob daltonismo).
- **Contatos virou diretório-CRM robusto** (ago/2026): tabela com empresa, telefone
  e nº de conversas; filtro por empresa; criar/editar/excluir. A ficha abre em duas
  colunas (identidade + conversas | anotações com teto e scroll) e a **empresa é
  clicável** — view da empresa com seus contatos, tudo como troca de view no mesmo
  modal (sem empilhar). "Abrir conversa" pede o número de propósito (evita o errado).
- **Tema Laranja + Verde + Branco** (ago/2026, troca do roxo): fundo neutro e
  chapado (aurora removida), escuro grafite/preto; a marca tem variante acessível
  por tema, validada por cálculo de contraste.

## Infra

Slug `navehub` · app `navehub-app` na `4030` · banco `navehub-db` na `5030`.
Compose igual em dev/prod, migrations em container no boot. Chassi e mapa de
portas em [[Infra]].

## Stack

Next 16 (App Router) · React 19 · Postgres 17 · Tailwind v4 · `pg` (sem ORM,
SQL direto) · TanStack Query · lucide · sonner.

## Decisões importantes

- **Atendimento + CRM juntos**, ao contrário do navetalks (só atendimento). O CRM
  que faltava lá — empresa como entidade, contato rico — é razão de existir aqui.
- **Fila como unidade de permissão** do atendimento, não cargo global: o acesso
  segue a associação pessoa↔fila. **Supervisor é papel escopado à fila**
  (`fila_membro.supervisor`), não global — quem transfere/gerencia o setor. A
  transferência, porém, pode cruzar setor (o supervisor manda pra qualquer usuário);
  o número não muda, a posse sim. Aplica [[Escopo de dado se clampa no servidor, num funil só]].
- **Fila compartilhada com posse explícita** (modelo Gestta): espera → puxar/enviar
  vira posse → finalizar/devolver solta. Leitura aberta no setor, escrita do dono.
- **SQL direto com `pg`**, não Prisma (o navetalks usava Prisma) — mesma escolha
  do Navetech Hub/Nexo.
- **WhatsApp atrás de adapter**: Cloud API oficial como default, mas o app fala
  com uma interface de canal, não com o provider.

## Aprendizados (viraram notas)

Só links; o texto mora na nota de princípio/técnica.

- [[Adapter de canal isola o app do provider de mensageria]]
- [[Permissão se valida no servidor, não na interface]]
- [[Sessão opaca no banco separa autenticação de permissão]]
- [[Permissão composta por papéis somados, não exceção por usuário]]
- [[Entidade núcleo cresce por tabela satélite, não por coluna]]
- [[Persistir a mensagem não espera a entrega, a entrega é status]]
- [[Sistema de cores e tema do dashboard]]
- [[Cor de marca precisa de variante acessível por tema]]
- [[Modal com conteúdo que cresce tem teto de altura e área que rola]]
- [[Componente de terceiro que usa Context não roda em Server Component]]
- [[Runner de migration em SQL puro dispensa o CLI do ORM]]
- [[Numeric e bigint do Postgres chegam como string no driver pg]]
- [[Navegação de dois níveis - trilho de produto e sidebar de contexto]] (o rail de
  ícones dirigido por catálogo já é o nível 1; a sidebar de contexto entra quando virar suíte)
- [[Primitivos, reaproveitamento e modularidade vêm antes da escala]] (pensamento)

## Próximos passos

- [ ] Autenticação real (substitui o fallback de dev em `sessao.ts`) — reusar
      [[Sessão opaca no banco separa autenticação de permissão]].
- [ ] Integração WhatsApp Cloud API: webhook de entrada + gancho de saída +
      credenciais por fila, via [[Adapter de canal isola o app do provider de mensageria]].
- [ ] Tempo real na inbox (LISTEN/NOTIFY ou SSE).
- [x] CRM base: **módulo Contatos** robusto (tabela, filtro por empresa, CRUD,
      ficha em duas colunas) + **painel** operacional na home. **feito** (ago/2026).
- [ ] CRM: empresa como **entidade** de verdade — hoje é texto e a "view da empresa"
      agrupa contatos por esse texto; falta tabela `empresa`, ficha própria e o
      módulo Empresas (rail). Contato rico (honorário, etiquetas via satélite), relatórios.
- [ ] Configurar remote e primeiro push.

## Conexões
- Usa: [[Design]] · [[Infra]] · [[Backend]]
- Substitui: [[navetalks]]
- Mapa: [[Projetos]]
