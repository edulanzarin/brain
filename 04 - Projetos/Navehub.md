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

Fundação de pé e verificada (build de produção limpo, typecheck limpo, `/inbox`
em 200 SSR contra o Postgres seedado):

- **Caixa de entrada de 3 colunas** (filas / lista de conversas / conversa /
  detalhes), no visual herdado do design system. Puxar da fila, responder,
  transferir e resolver por server actions.
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
