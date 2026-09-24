---
tags: [tipo/atomica, camada/padrao, dev/backend, sql]
criado: 2026-09-24
---

# Fila no Postgres entra na transação do estado, e o NOTIFY só acorda no commit

> Efeito externo que precisa acontecer depois de uma mudança de estado (mandar o
> convite depois do pagamento) vira linha numa tabela de tarefas, gravada na
> MESMA transação da mudança. Se a rede cair, a tarefa continua lá. E o
> `pg_notify` feito dentro da transação só é entregue no commit: o trabalhador
> acorda quando o dado já é visível, nem antes nem nunca.

## O problema

Chamar a API externa dentro da transação segura a linha travada enquanto a rede
responde. Chamar depois do commit perde o efeito se o processo cair entre um e
outro: o pedido fica pago e o convite nunca sai, sem erro em lugar nenhum.

## A solução

```sql
create table tarefa (
  id bigserial primary key, tipo text not null, dados jsonb not null,
  chave text, situacao text not null default 'pendente',   -- pendente | feita | morta
  tentativas int not null default 0, max_tentativas int not null default 8,
  executar_em timestamptz not null default now(), travada_ate timestamptz, ultimo_erro text
);
-- uma tarefa PENDENTE por chave: enfileirar duas vezes a mesma entrega não duplica
create unique index on tarefa (chave) where situacao = 'pendente' and chave is not null;
```

- **Enfileirar**: `insert ... on conflict (chave) where situacao = 'pendente' ... do nothing`
  e `select pg_notify('tarefa', '')`, os dois com o executor da transação.
- **Reservar**: `update ... set travada_ate = now() + '2 min', tentativas = tentativas + 1
  where id in (select id ... for update skip locked limit N) returning *`. Quem caiu
  no meio devolve a tarefa sozinho quando a trava vence.
- **Falhar**: o destino depende do tipo de erro. Recusa do outro lado (bot
  bloqueado, sem permissão) morre na hora; limite (429) espera o que o outro lado
  mandou, sem gastar tentativa; falha de rede tenta de novo com espera crescente.
  Tarefa que morre chama um `aoDesistir` que deixa a falha visível para o dono.
- **Executar relendo a fonte**: entre enfileirar e executar a pessoa pode ter
  renovado ou saído. O executor confere o estado atual antes de agir.

O trabalhador pode morar no próprio processo do app (no Next, pelo
`instrumentation.ts`), com uma conexão LISTEN que também serve outros canais.

## O que mais vale lembrar

- **Teste de integração disputa a fila com qualquer trabalhador vivo no mesmo
  banco.** Com o app de pé, o NOTIFY acorda o trabalhador dele antes de o teste
  drenar a fila: o efeito acontece, mas no outro processo e depois da
  conferência, e o teste falha dizendo que o convite não existe. Rode a
  integração contra banco sem app ligado.
- A chave de deduplicação costuma levar o que distingue a ocorrência
  (`remover:<membro>:<vencimento>`), para a remoção de um vencimento novo não
  colidir com a tarefa morta do anterior.
- Tarefa morta por recusa volta a ser tentada só depois de um intervalo longo
  (horas), e não a cada rodada: insistir contra um "não" só enche o log.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]] ·
  [[Guarde a intenção e o processo se reconstrói dela]]
- Irmã: [[Persistir a mensagem não espera a entrega, a entrega é status]] ·
  [[Recusa não é falha; contra o não do servidor, insistir é ruído]] ·
  [[Webhook de dinheiro precisa de duas travas, a do evento e a do efeito]] ·
  [[Agenda recorrente é um serviço do compose, não um crontab do host]]
- Visto em: [[telebot]]
- Mapa: [[Dados]] · [[Backend]]
