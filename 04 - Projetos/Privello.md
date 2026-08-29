---
tags: [tipo/projeto, projeto/privello]
criado: 2026-08-29
status: ativo
codigo_em: ~/Dev/privello
---

# Privello

> Classificado de acompanhantes por cidade, com verificação de documento antes
> da publicação. Planos para quem anuncia (teto de mídia, story e destaque) e um
> passe VIP para quem procura. Domínio já registrado; a referência de mercado é
> o FatalModel, e a segunda metade — venda de conteúdo por assinatura de perfil,
> tipo Privacy — fica para o corte seguinte.

Código em: `~/Dev/privello`

## Estado atual

**Primeiro corte fechado (ago/2026).** Vitrine pública, painel de quem anuncia e
moderação, rodando no compose e com build de produção passando. Sete commits na
`main`, sem remote ainda.

Ponta a ponta exercitado: cadastro, criação do anúncio, envio de documento,
aprovação na moderação, publicação, upload de mídia dentro do teto do plano,
story com prazo de 24 h, compra de plano e de VIP (provedor simulado),
avaliação, denúncia e suspensão. `npm run verificar` cobre 27 regras contra o
banco de dev.

## Infra

Slug `privello` · app `privello-app` na `4075` · banco `privello-db` na `5075`.
Compose com app e db, imagem standalone, migrations e seed no entrypoint, mídia
em volume próprio. Chassi e mapa de portas em [[Infra]].

## Stack

Next.js 16 (App Router, Server Components, Server Actions) · React 19 ·
TypeScript · Postgres 16 com Prisma 7 · Auth.js v5 (credenciais, JWT) ·
Tailwind v4 (tokens em `@theme`, classes em `@layer components`).

## Decisões importantes

- **Papel é permissão, não conta separada.** Todo mundo entra como `CLIENTE`;
  abrir anúncio vira `ACOMPANHANTE` no mesmo login. Quem anuncia também assina o
  VIP, e nenhum dos dois lados precisa de segundo cadastro.
- **Nenhum caminho leva o anúncio de rascunho ao ar.** Ele passa obrigatoriamente
  por `EM_ANALISE`, e quem publica é a moderação depois de conferir documento com
  selfie. É a trava dos 18 anos, e ela é estrutural: não existe transição direta
  no código.
- **O passe VIP destrava três coisas, e só três**: story marcado como exclusivo,
  contato de quem escolheu atender só assinante, e as avaliações (ler e
  escrever). Plano único, o que muda é a duração. As três checagens moram no
  servidor — quando o contato é fechado, o telefone não chega ao HTML.
- **A receita é espaço de anúncio e assinatura, nunca percentual sobre
  encontro.** Não existe campo de comissão no schema, e a ausência é deliberada:
  cobrar pelo anúncio é classificado, comissionar o programa é outra natureza de
  serviço.
- **Documento de verificação não divide nada com a mídia pública** — outro model,
  outra pasta, outra rota, aberta só para a moderação.
- **Cidade não vira página só por estar no cadastro.** A rota, o índice e o
  sitemap saem da mesma consulta de "onde há anúncio ativo" —
  [[A superfície indexável sai da mesma consulta que o conteúdo]].
- **Pagamento nasce como fronteira.** O simulado existe porque Stripe, Mercado
  Pago e PagSeguro proíbem conteúdo adulto em contrato: o fluxo de venda inteiro
  precisava existir antes de haver adquirente —
  [[Provedor de pagamento entra por interface, e o simulado é a primeira implementação]].
- **Estética própria, escura, celular antes de tudo.** Não herda o dialeto de
  dashboard dos sistemas internos; o público é visitante de telefone, não
  operador — [[Estética é por projeto, princípio de design é que se reusa]].
- **Expiração de assinatura se apura na leitura** do painel e do admin, sem
  agendador — [[Rotação por período se apura na leitura, e dispensa agendador]].

## Aprendizados (viraram notas)

- [[A superfície indexável sai da mesma consulta que o conteúdo]]
- [[Página que consulta o banco não pode nascer no build]]
- [[Portão de conteúdo cobre a tela, não o HTML]]
- [[Linha no banco não garante o arquivo no disco]]

## Próximos passos

- [ ] **Gateway real.** É o bloqueio comercial, não técnico: precisa de PIX
      direto por PSP que aceite o segmento, ou adquirente high-risk. A interface
      `ProvedorPagamento` já está pronta para receber.
- [ ] Assinatura por perfil (a metade "Privacy"): feed pago da acompanhante, com
      repasse. É o segundo corte combinado.
- [ ] Mídia em armazenamento de objeto (R2 ou Backblaze) implementando
      `Armazenamento` — disco local não passa de uma máquina.
- [ ] Revisão jurídica de termos e privacidade, e definição do encarregado de
      dados da LGPD.
- [ ] Redimensionar imagem no upload; hoje o arquivo original é servido como veio.
- [ ] Busca por nome e filtro de faixa de preço na listagem da cidade.

## Conexões
- Usa: [[Design]] · [[Infra]]
- Mapa: [[Projetos]]
