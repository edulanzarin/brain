---
tags: [tipo/projeto, projeto/monofire]
criado: 2026-08-28
status: ativo
codigo_em: ~/Dev/monofire
---

# monofire

> Marketplace de cursos de jogos competitivos — um "Hotmart de game". Quem sabe jogar
> publica o curso, quem quer subir compra e assiste. Foco em League of Legends, com
> Valorant, CS2, TFT e Dota 2 no mesmo catálogo.

Código em: `~/Dev/monofire`

## Estado atual

Marketplace completo rodando local, build de produção passando e fluxo exercitado ponta
a ponta: catálogo com filtro na URL, página de venda com prévia gratuita, compra
(pedido → cobrança → matrícula), área do aluno com player e progresso por aula, e
estúdio do criador com módulos, aulas, preço e publicação. Pagamento é o provedor
simulado — aprova na hora, sem credencial. Sem remote git.

## Infra

Slug `monofire` · app `monofire-app` na `4074` · banco `monofire-db` na `5074`.
Compose com app e db, imagem standalone, migrations e seed no entrypoint.
Chassi e mapa de portas em [[Infra]].

## Stack

Next.js 16 (App Router, Server Components, Server Actions) · React 19 · TypeScript ·
Postgres 16 com **Prisma 7** · Auth.js v5 (credenciais, JWT) · Tailwind v4 (tokens em
`@theme`, classes em `@layer components`).

## Decisões importantes

- **Papel é permissão, não conta separada.** Todo mundo se cadastra como `ALUNO`; virar
  `CRIADOR` é abrir o estúdio, com a mesma conta e o mesmo login. Um marketplace onde
  quem vende também compra não pode pedir dois cadastros.
- **Pagamento nasce como fronteira, não como integração.** O primeiro corte usa provedor
  simulado justamente para o fluxo de venda existir inteiro antes de existir gateway —
  [[Provedor de pagamento entra por interface, e o simulado é a primeira implementação]].
  Mercado Pago é o candidato natural quando for pra valer (pix + cartão, público BR).
- **Acesso é tabela própria.** `matricula` separada de `pedido`, com o ponteiro
  opcional, porque curso gratuito e cortesia não têm venda —
  [[Acesso comprado é linha própria, não status do pedido]].
- **A regra da compra mora fora da server action**, em `src/lib/compra.ts`, e é
  exercitada por `npm run verificar` contra o banco de dev —
  [[A regra que a server action executa mora fora dela]].
- **Cadeado no servidor, sempre.** Matrícula, dono do curso e aula de amostra são
  conferidos em toda página e action; a URL da aula é adivinhável, então a checagem não
  pode viver na tela que lista — [[Permissão se valida no servidor, não na interface]].
- **Estética própria, escura, sem tema claro.** Não herda o dialeto de dashboard dos
  sistemas internos: aqui o público é comprador, não operador. O que se reusa é o
  princípio (token semântico, escala fechada, hierarquia por superfície), não a cara —
  [[Estética é por projeto, princípio de design é que se reusa]].
- **Curso nasce rascunho e só publica com aula dentro.** Publicar caixa vazia é o jeito
  mais rápido de um marketplace novo parecer abandonado.
- **Vídeo é URL, não provedor cadastrado.** A aula guarda o link e a tela decide como
  tocar (YouTube ou arquivo), então colar um link do YouTube funciona sem campo de
  "tipo de vídeo" no formulário.

## Aprendizados (viraram notas)

- [[Provedor de pagamento entra por interface, e o simulado é a primeira implementação]]
- [[Acesso comprado é linha própria, não status do pedido]]
- [[A regra que a server action executa mora fora dela]]

## Próximos passos

- [ ] Gateway real (Mercado Pago: pix + cartão) implementando `ProvedorPagamento`, com
      webhook levando o pedido de `PENDENTE` a `PAGO`.
- [ ] Repasse ao criador: hoje a receita do estúdio é bruta, sem comissão da plataforma.
- [ ] Upload de vídeo próprio (hoje só URL) e capa hospedada.
- [ ] Página pública do criador, com todos os cursos dele.
- [ ] Reordenar módulos e aulas no estúdio (hoje a ordem é a de criação).

## Conexões
- Usa: [[Design]] · [[Infra]]
- Mapa: [[Projetos]]
