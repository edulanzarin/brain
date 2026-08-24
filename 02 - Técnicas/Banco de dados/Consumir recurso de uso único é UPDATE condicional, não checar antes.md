---
tags: [tipo/atomica, camada/padrao, dev/backend, sql]
criado: 2026-08-11
---

# Consumir recurso de uso único é UPDATE condicional, não checar antes

> Um cupom que vale uma vez, uma vaga, um estoque de um — não se resgata com
> "SELECT pra ver se está livre, depois grava". Funde-se no **UPDATE condicionado
> ao estado livre** e deixa-se o `rowCount` decidir quem ganhou.

## O problema

O reflexo é modelar o resgate em dois passos: consultar se o recurso está
disponível, e então marcá-lo como usado. Entre a consulta e a marcação existe uma
janela — dois pedidos concorrentes com o mesmo código consultam "livre" ao mesmo
tempo, e os dois seguem em frente. O resultado não é erro barulhento: são duas
vagas liberadas por um cupom de uso único, aparecendo só quando alguém conta.

## A solução

A verificação e a mudança viram **uma operação só**, que o próprio banco
serializa. O `WHERE` carrega a condição de disponibilidade; quem recebe a linha de
volta é o vencedor da corrida:

```sql
UPDATE coupons
   SET redeemed_by = $2, redeemed_at = now()
 WHERE code = $1 AND redeemed_at IS NULL
```

`rowCount = 1` → resgatei, o recurso é meu. `rowCount = 0` → não existe ou já foi
consumido; trato como inválido. Sem transação explícita, sem `SELECT ... FOR
UPDATE`, sem lock de aplicação — a atomicidade do próprio UPDATE basta.

## O que mais vale lembrar

- **O UPDATE não devolve só quem ganhou: devolve o dado que decide o resto.** Um
  `RETURNING` no mesmo comando traz, junto com a vitória na corrida, o valor
  guardado no recurso — no Evento Navecon o cupom passou a carregar a
  porcentagem de desconto, e é ela que diz se a inscrição pula o checkout (100%)
  ou só sai mais barata. Ganho e parâmetro chegam numa ida só, sem um SELECT
  depois que reabriria a janela.
- **Consuma antes de gravar o dependente, e compense se a gravação falhar.**
  Quando o consumo decide o fluxo, ele tem que vir primeiro — gravar antes
  obrigaria a adivinhar. A ordem inverte quem limpa a sujeira: em vez de apagar a
  linha órfã quando o consumo perde a corrida, devolve-se o uso
  (`SET uses = uses - 1 WHERE uses > 0`) quando a gravação falha. O Evento Navecon
  fazia o primeiro e passou ao segundo ao ganhar cupom com porcentagem. As duas
  formas funcionam; a que compensa é a que sobrevive ao consumo virar parâmetro.
- **O mesmo padrão faz o "claim" de efeito colateral idempotente.** Para não
  mandar o mesmo e-mail duas vezes quando dois gatilhos (webhook + poller)
  concorrem, reivindique o direito de enviar antes de enviar:
  `UPDATE ... SET notificado = true WHERE id = $1 AND notificado = false
  RETURNING id` — só quem recebeu a linha dispara. É a mesma corrida, resolvida no
  mesmo lugar. Ver [[Polling substitui webhook quando não há IP público]].
- Vale para **transição de estado disparada uma vez** (livre→usado,
  pendente→enviado), não para "no máximo uma linha" pura — essa é `unique` +
  `on conflict do nothing`.
- **Generaliza para limite N** trocando só a condição: um cupom que vale N vezes é
  `UPDATE ... SET uses = uses + 1 WHERE code = $1 AND uses < max_uses` — a corrida
  pelo último uso continua decidida pelo `rowCount`, sem lock. Uso único é o caso
  N=1 (no Evento Navecon o cupom passou de `redeemed_at IS NULL` para esse contador
  sem mudar a ideia).

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[Uma resposta canônica de um grupo é um token compartilhado]]
- Visto em: [[Evento Navecon]]
- Mapa: [[Dados]]
