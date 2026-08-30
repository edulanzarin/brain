---
tags: [tipo/atomica, camada/padrao, armadilha, dev/frontend]
criado: 2026-08-30
---

# Rascunho no navegador leva o dono na chave

> Guardar o formulário no `localStorage` resolve fechar a aba. Cria outro
> problema no mesmo instante: `localStorage` é do NAVEGADOR, e conta é de
> pessoa. Trocar de login abre o cadastro com os dados de quem estava antes.

## O problema

A chave nasce genérica, porque na hora de escrever só existe uma pessoa em
mente:

```ts
localStorage.setItem("app.rascunho", JSON.stringify(r));
```

Aí alguém sai, entra com outra conta, e o formulário aparece preenchido. Não é
erro de leitura: o dado está lá, é do navegador, e o navegador é o mesmo.

O tamanho do estrago depende do que o formulário guarda. Num cadastro de
anúncio onde o nome público NÃO é o nome de registro, o campo vem preenchido
com algo plausível — a pessoa avança sem reler e publica o dado de outra.

## A correção

O dono entra na chave, e vem do servidor por propriedade — nunca de algo que o
próprio navegador guardou:

```ts
const chave = `app.rascunho.${usuarioId}`;
```

E com ele **o cache e os ouvintes**, quando o rascunho é lido por
`useSyncExternalStore`: `getSnapshot` devolve a mesma referência enquanto nada
muda, então um cache único devolveria o objeto da conta anterior até a primeira
escrita — o mesmo defeito com outra roupa.

```ts
const caches = new Map<string, Rascunho>();
const ouvintes = new Map<string, Set<() => void>>();
```

## O que fica dito na tela

Rascunho de navegador não segue a pessoa para outro aparelho, e isso não é
defeito — é a escolha de não criar registro pela metade no banco. Mas precisa
estar escrito onde ela preenche, e não descoberto quando ela abre o celular.

## Conexões
- Princípio: [[Estado mutável se lê da fonte no uso, não de cópia guardada]] — a
  cópia no navegador é de UMA sessão, e tratá-la como global é o erro.
- Irmã: [[Sem servidor, a migração de dado local acontece na leitura]] ·
  [[Sessão de outro domínio só se injeta rodando na origem dele]]
- Visto em: [[Privello]]
- Mapa: [[Frontend]]
