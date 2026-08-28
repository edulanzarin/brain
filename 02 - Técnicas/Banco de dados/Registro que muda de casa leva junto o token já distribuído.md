---
tags: [tipo/atomica, camada/padrao, sql, dev/backend]
criado: 2026-08-28
---

# Registro que muda de casa leva junto o token já distribuído

> Reorganizar tabelas é assunto interno; o link que já está na caixa de entrada de
> alguém não é. Na migration que muda o dono do registro, o identificador que o
> mundo lá fora segura vem junto — senão a mudança de schema desliga o que já foi
> prometido.

## O problema

Promover um modo escondido a entidade própria costuma vir com migration que
recria as linhas no destino. Se o destino gera identificadores novos, tudo bate no
banco e nada bate fora dele: o link `/f/<token>` que saiu por e-mail semana
passada passa a responder "link inválido", e quem recebeu não tem como saber que
o problema é seu. O mesmo vale para protocolo de atendimento, número de pedido e
qualquer chave que já virou texto na mão de terceiro.

## A solução

Copiar o token junto, e deixar a migration inteira dizer isso em SQL — nada de
script de fora que alguém esquece de rodar.

```sql
-- 1. a casa nova nasce com o token da antiga: link já enviado continua valendo
insert into rh_desempenho (rodada_id, codigoempresa, codigofunccontr, token, status)
select d.envio_id, d.codigoempresa, d.codigofunccontr, d.token, d.status
  from envio_destinatario d
 where d.codigofunccontr is not null;

-- 2. a resposta única da casa antiga vira a primeira das N da nova, e o jsonb
--    embrulhado ({"valores": {...}}) entra no formato plano do destino
insert into rh_desempenho_resposta (desempenho_id, respondido_por_nome, valores)
select a.id, coalesce(nullif(btrim(d.respondido_por_nome), ''), 'Não informado'),
       coalesce(d.valores -> 'valores', '{}'::jsonb)
  from envio_destinatario d
  join rh_desempenho a on a.token = d.token   -- o token é a ponte entre as duas
 where d.codigofunccontr is not null and d.status = 'respondido';

-- 3. só então a casa antiga é esvaziada e as colunas mortas caem
delete from envio_destinatario where codigofunccontr is not null;
alter table envio_destinatario drop column codigofunccontr, drop column classiforgan;
```

O token preservado também é a **junção da própria migration** (passo 2): sem ele
seria preciso carregar o id gerado linha a linha para reencontrar a origem.

## O que mais vale lembrar

- **Deixar de existir é diferente de deixar de valer.** Configuração de outra
  pessoa que perde sentido com a mudança — uma regra de envio automático que só
  existia no modo antigo — se **desliga** (`ativo = false`), não se apaga: some do
  caminho sem sumir da vista de quem a criou.
- **Limpar o schema faz parte da mesma migration.** Coluna que ficou sem uso e
  `not null` que a feature antiga tinha relaxado voltam ao lugar no mesmo passo;
  senão ninguém sabe mais se aquela coluna ainda alimenta alguma coisa.
- **Testar com dado semeado antes de rodar em produção.** O banco de dev costuma
  estar vazio justamente do caso que a migration migra: semeie o formato antigo,
  rode, e confira os dois lados — o que veio e o que ficou intacto.

## Conexões
- Princípio: [[Migração de dados mantém o antigo como reserva até a virada]] ·
  [[O que tem ciclo de vida próprio é entidade própria, não modo de outra]]
- Irmã: [[Formulário público por token opaco fica fora do gate de sessão]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Dados]]
