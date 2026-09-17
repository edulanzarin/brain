---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-09-17
---

# Webhook de dinheiro precisa de duas travas, a do evento e a do efeito

> Provedor de pagamento reentrega notificação **por desenho**, não por defeito.
> Guardar o id do evento recebido resolve a repetição idêntica; não resolve dois
> eventos DIFERENTES falando do mesmo pagamento. A segunda trava é no efeito, e
> ela mora no banco.

## O problema

A primeira defesa é óbvia e insuficiente:

```sql
insert into evento_gateway (gateway, evento_id) values ($1, $2)
on conflict do nothing returning evento_id
```

Veio linha, é a primeira vez; não veio, já processei. Isso corta a reentrega
literal, que é o caso comum.

Só que o provedor manda **vários eventos sobre a mesma cobrança** — criada, em
análise, aprovada — e alguns não têm id de evento estável (o Mercado Pago, por
exemplo, obriga a compor a chave com o id do pagamento mais a ação). Dois eventos
com ids diferentes apontando para o mesmo pagamento passam pela primeira trava
lado a lado, e cada um credita.

O modo de falha é o pior possível: não dá erro. O criador recebe o dobro na
carteira, o comprador recebe dois convites, e a divergência só aparece no
fechamento.

## A solução

A segunda trava cerca o EFEITO, e é o banco que a garante —
[[Um invariante se garante na estrutura, não no processo]]:

```sql
-- um pedido pago gera um acesso, e só um
alter table acesso add constraint acesso_pedido_unico unique (pedido_id);

-- um pedido credita a carteira uma vez
create unique index movimento_venda_unico_idx
  on movimento_carteira (pedido_id) where tipo = 'venda';
```

E a transação que confirma lê o pedido com `for update` antes de decidir:

```sql
select status from pedido where id = $1 for update
```

Duas notificações no mesmo instante entram em FILA nessa linha, em vez de as duas
lerem "aguardando". A segunda acorda com o status já em `pago` e sai sem
escrever. O `unique` é a rede embaixo disso: se algum caminho novo esquecer a
checagem, ele recusa em vez de duplicar.

## O que mais vale lembrar

- **Confirme a situação com o provedor antes de entregar.** O evento diz "algo
  mudou", não "foi aprovado". Consultar a cobrança evita entregar acesso no
  evento errado.
- **A entrega fica FORA da transação.** Falar com o Telegram (ou com quem for)
  pode demorar e falhar, e segurar transação aberta trava a linha do pedido para
  todo mundo. O preço é o acesso existir alguns segundos sem o convite, então a
  entrega precisa ser **repetível** — no telebot, `/meuacesso` e o cron refazem
  essa parte. O que não pode é o acesso não existir.
- **Notificação com assinatura inválida responde 401, não 200.** Engolir como
  "ok" apaga a tentativa de fraude do log do provedor.
- O espelho do saldo (`conta.saldo_centavos`) e o movimento do extrato escrevem
  na MESMA transação. A tabela de movimento é a verdade; o campo existe só para a
  tela não somar o extrato inteiro a cada abertura.

## Como conferir que está valendo

Mande três notificações: a mesma duas vezes, e uma terceira com id de evento
novo apontando para o mesmo pagamento. O saldo pode subir uma vez só. No telebot
isso deu 129786 → 134577 (+4791, o líquido de uma venda de R$ 49,90 com taxa de
3,99%) e parou aí.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]] · [[Fornecedor externo entra pelo contrato do app, não o app pelo dele]]
- Irmã: [[A assinatura autentica o dado, não quem o trouxe]] · [[Chamada externa tem timeout e erro tratado]]
- Visto em: [[telebot]]
- Mapa: [[Backend]]
