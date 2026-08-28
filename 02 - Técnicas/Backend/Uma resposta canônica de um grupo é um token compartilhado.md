---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-07-28
---

# Uma resposta canônica de um grupo é um token compartilhado

> Quando o avaliador é "o setor" e não uma pessoa fixa, emita **um** token para o
> grupo e mande o **mesmo link a todos** — não um destinatário por pessoa. Quantas
> respostas esse link aceita é decisão à parte: uma canônica (a primeira fecha) ou
> uma por gestor, distinguidas pelo nome de quem responde.

## O problema

Um departamento tem N gestores (supervisor + coordenador). Você quer UMA avaliação
sobre o colaborador, respondida por quem estiver disponível. Se emitir um link por
gestor, ganha N respostas possíveis e a ambiguidade de qual conta — e dois gestores
podem responder o mesmo caso em versões diferentes.

## A solução

Modelar a avaliação como **um recurso** (uma linha, um token), não como uma lista
de pessoas. Os destinatários reais só aparecem no **disparo**: resolve os e-mails
dos gestores do setor na hora e manda o mesmo link a todos. A submissão fecha o
recurso (`status = respondido`), então o segundo gestor que abrir o link vê "já
respondido". É o mesmo mecanismo do token opaco ([[Formulário público por token opaco fica fora do gate de sessão]]), só que a credencial pertence ao GRUPO.

```
-- uma linha por avaliação; email pode ser null (o alvo é o setor, não uma pessoa)
insert into envio_destinatario (envio_id, token, classiforgan, funcionario_nome, ...)

-- no disparo: os destinatários saem do setor, um link para todos
const para = await gestoresDoSetor(classiforgan);   // ['a@x', 'b@x']
await enviarEmail({ para, assunto, html: linkUnico });
```

## O que mais vale lembrar

- **Uma resposta ou várias é escolha do domínio, não do schema.** Avaliação de
  experiência quer UMA (a decisão do setor sobre o marco) e o primeiro fecha —
  invariante que se garante na estrutura, não na expectativa de que só um gestor
  responda: [[Um invariante se garante na estrutura, não no processo]]. Avaliação de
  desempenho quer VÁRIAS, uma por gestor: mesmo token compartilhado, resposta em
  tabela filha, e o link fecha por ato explícito do RH — não na primeira submissão.
- **Reusar a tabela de destinatários com discriminador nullable envelheceu mal.**
  Foi o que se fez em jul/2026 (a linha de colaborador com `email` null dentro de
  `envio_destinatario`) e foi revertido em ago/2026: o hóspede tinha ciclo de vida
  próprio — várias respostas, repetição no tempo, fim por outro evento — e o modo
  escondido cobrou isso em tela sem filtro e resposta perdida. Compartilhar tabela
  vale enquanto o ciclo for o mesmo; quando diverge, é entidade própria
  ([[O que tem ciclo de vida próprio é entidade própria, não modo de outra]]).
- Índice único parcial evita o mesmo alvo duas vezes no mesmo disparo — mas ele é
  por `(disparo, alvo)`, nunca só por alvo: único por alvo proibiria a segunda
  avaliação da mesma pessoa e mataria o histórico.
- É a generalização de um fluxo já existente (a avaliação de experiência já
  mandava um token a todos os gestores do setor); o padrão nasceu ali e virou
  reutilizável quando o envio ad-hoc precisou do mesmo comportamento.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]] ·
  [[O que tem ciclo de vida próprio é entidade própria, não modo de outra]]
- Irmã: [[Formulário público por token opaco fica fora do gate de sessão]] ·
  [[Registro que muda de casa leva junto o token já distribuído]]
- Visto em: [[Navetech Hub]] · [[navetalks]]
- Mapa: [[Backend]]
