---
tags: [tipo/moc]
criado: 2026-07-20
---

# Frontend

Técnicas e armadilhas presas ao React e ao Next. É a camada mais volátil do cérebro —
quando a ferramenta muda, estas notas envelhecem, e tudo bem. O que precisa sobreviver
mora em [[Base]].

Stack atual: Next.js (App Router) · React · TypeScript · Tailwind.

## Armadilhas do Next

- [[router.replace do Next falha no build de produção]] — funciona em `dev`, falha
  calado em `build`. Princípio: [[Verificar no build de produção, não só em dev]].
- [[Componente de ícone não atravessa a fronteira server-client]]
- [[React reseta o formulário ao fim de uma Server Action]]

## Estado e renderização

- [[Cache do React Query não é lugar de estado de interface]]
- [[Portal condicional dispensa o flag de montagem]]

Princípios: [[Estado compartilhável mora na URL]] ·
[[Estado de tela pertence à seção, não à página]]

## TypeScript

- [[NoInfer faz o genérico sair da lista, não do valor padrão]]

## Fronteira com o Design

O que é **aparência** (token, componente, estado visual) mora em [[Design]]. O que é
**comportamento do framework** mora aqui. Na dúvida: se continua verdade trocando o
Tailwind por outro CSS, é Design; se depende do React, é Frontend.

---

Voltar para [[Início]]
