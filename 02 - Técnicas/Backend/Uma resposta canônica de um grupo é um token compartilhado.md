---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-07-28
---

# Uma resposta canônica de um grupo é um token compartilhado

> Quando VÁRIOS podem responder mas só UMA resposta deve valer — o avaliador é
> "o setor", não uma pessoa fixa —, emita **um** token para o grupo e mande o
> **mesmo link a todos**; o primeiro que responde fecha. Não crie um destinatário
> por pessoa.

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
respondido". É o mesmo mecanismo do token opaco ([[Formulário público por token
opaco fica fora do gate de sessão]]), só que a credencial pertence ao GRUPO.

```
-- uma linha por avaliação; email pode ser null (o alvo é o setor, não uma pessoa)
insert into envio_destinatario (envio_id, token, classiforgan, funcionario_nome, ...)

-- no disparo: os destinatários saem do setor, um link para todos
const para = await gestoresDoSetor(classiforgan);   // ['a@x', 'b@x']
await enviarEmail({ para, assunto, html: linkUnico });
```

## O que mais vale lembrar

- **Reusar a tabela de destinatários com um discriminador nullable** em vez de
  criar tabela nova: a mesma `envio_destinatario` serve "e-mail solto" (fluxo
  antigo) e "sobre um colaborador" — a linha de colaborador tem `email` null e
  colunas de contrato preenchidas; o disparo escolhe o caminho por elas. Um caso
  especial vira uma variação de dado, não um schema paralelo.
- O "primeiro fecha" é o invariante — garanta-o na estrutura, não na expectativa
  de que só um gestor vá responder: [[Um invariante se garante na estrutura, não
  no processo]].
- Índice único parcial evita o mesmo alvo duas vezes no mesmo envio.
- É a generalização de um fluxo já existente (a avaliação de experiência já
  mandava um token a todos os gestores do setor); o padrão nasceu ali e virou
  reutilizável quando o envio ad-hoc precisou do mesmo comportamento.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[Formulário público por token opaco fica fora do gate de sessão]]
- Visto em: [[Navetech Hub]] · [[navetalks]]
- Mapa: [[Backend]]
