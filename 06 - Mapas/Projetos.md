---
tags: [tipo/moc]
criado: 2026-07-20
---

# Projetos

Mapa das **aplicações** — os sistemas onde a base é aplicada. O conhecimento em si não
mora aqui: mora em [[Base]] (princípios) e em [[Design]], [[Frontend]], [[Backend]],
[[Dados]] e [[Infra]] (técnicas).

## Ativos

- [[Questor Hub]] — dashboard fiscal e contábil sobre o banco do ERP.
- [[Navecon Controller]] — automações e integrações.
- [[Navedesk]] — chamados internos.
- [[Cofre Digital]] — certificados, acessos e alvarás da intranet.
- [[Evento Navecon]] — landing da imersão com inscrição e pagamento (Mercado Pago).

## Regra: projeto não linka projeto

Dois sistemas que não trocam dado **não se linkam**, mesmo que compartilhem stack,
visual ou servidor. O que eles têm em comum não é um o outro — é a base.

Errado: `Navedesk — Reusa o visual de: [[Questor Hub]]`
Certo: `Navedesk — Usa: [[Design]]`

O primeiro cria uma ponte falsa: sugere dependência onde não existe, faz o projeto mais
antigo virar dono do conhecimento e embola o grafo num novelo. O segundo diz a verdade —
os dois bebem da mesma fonte, e nenhum depende do outro. Ver
[[Conhecimento pertence à base, não ao projeto]].

Um projeto só linka outro se houver **relação real**: consome a API do outro,
compartilha banco, ou um substituiu o outro.

## Onde procurar o conhecimento

| Pergunta | Mapa |
|---|---|
| Por que a regra é essa? | [[Base]] |
| Como fica a tela? | [[Design]] |
| React ou Next fazendo coisa estranha? | [[Frontend]] |
| API, arquivo, integração? | [[Backend]] |
| Query pesada, modelagem? | [[Dados]] |
| Como sobe o projeto? | [[Infra]] |
| Onde está esse dado no ERP? | [[Banco Questor]] |

## Começando um projeto novo

1. **Infra primeiro** — slug, par de portas, compose: o chassi de [[Infra]].
2. **Design em seguida** — tokens e escala antes da primeira tela; checklist do [[Design]].
3. **Nota de projeto** em `04 - Projetos` pelo template [[Projeto]], com o caminho do
   código e a tag `#projeto/<slug>` (só nesta nota).
4. **Cor no grafo** — novo `colorGroup` em `graph.json`; regras no `CLAUDE.md`.

## Ao aprender algo no meio do caminho

O aprendizado **não fica na nota do projeto**. Sobe pra camada certa na hora, e o
projeto só linka — critério em
[[Camadas do conhecimento - princípio, padrão, aplicação]].

---

Voltar para [[Início]]
