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

- **Caixa de entrada de 3 colunas** (filas / lista de conversas / conversa), com a
  **ficha do contato em modal** (não coluna fixa). Design Aurora Glass roxo, denso,
  dois níveis de vidro (o overlay opaco é a base dos modais). Assumir, responder
  (envio **otimista** com `useOptimistic`), transferir e **finalizar** por server
  actions; filtros na URL; SLA por conversa (relógio colorido).
- **Modais** (pedido do Eduardo): ficha do contato (carrega notas via TanStack,
  edita, timeline de anotações), transferir (só membros da fila), nova conversa
  (picker de contato), respostas salvas, e confirmação do finalizar.
- **Núcleo de permissão = fila** (número de WhatsApp cadastrado como caixa). Quem
  vê/responde uma conversa é MEMBRO da fila (`fila_membro`), ou admin do
  escritório; transferência só vai para outro membro da mesma fila. Tudo trancado
  no servidor. Verificado com o exemplo Geral/DP: Bruno (só Geral) não enxerga
  conversa do DP; Carla (membro do DP) enxerga.
- **Multi-tenant por `org`**: toda linha carrega `org_id`.
- Envio ao WhatsApp é um **gancho** (`enviarAoProvedor`), ainda stub.
- Login ainda não existe: em dev o "usuário atual" vem de um cookie (seletor "Ver
  como") pra demonstrar a regra de fila ao vivo.

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
  segue a associação pessoa↔fila, e a transferência respeita a mesma fronteira.
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
- [ ] CRM: empresa como entidade, contato rico (aproveitar o que o navetalks já
      pensou de modelagem), relatórios.
- [ ] Configurar remote e primeiro push.

## Conexões
- Usa: [[Design]] · [[Infra]] · [[Backend]]
- Substitui: [[navetalks]]
- Mapa: [[Projetos]]
