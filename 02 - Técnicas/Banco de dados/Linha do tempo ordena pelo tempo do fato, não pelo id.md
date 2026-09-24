---
tags: [tipo/atomica, camada/padrao, sql, armadilha]
criado: 2026-09-24
---

# Linha do tempo ordena pelo tempo do fato, não pelo id

> `order by id desc` parece o mesmo que "mais recente primeiro" porque, no uso
> normal, a linha é gravada na hora em que o fato acontece. Basta uma gravação
> fora de ordem (semente, importação, reprocessamento, fila atrasada) para a
> linha do tempo mostrar "há 6 dias" acima de "há 2 horas".

## O problema

O id sequencial ordena a GRAVAÇÃO. A tela de atividade quer a ordem dos
ACONTECIMENTOS. As duas coincidem por acaso, e o acaso se desfaz sem aviso:

- a semente que grava cliente por cliente, cada um com a própria história;
- a importação de um sistema antigo;
- a tarefa da fila que registra o evento minutos depois, atrás de outras.

No painel do telebot, o feed da conta de demonstração saiu com datas de agosto
entre eventos de hoje. Nenhum erro, só uma linha do tempo que não é uma.

## A solução

Ordenar pela coluna do fato, com o id como desempate:

```sql
order by criada_em desc, id desc
```

E conferir o que depende da ordem antiga. No feed ao vivo, a regra "o que chegou
pelo stream é mais novo que a lista inicial" comparava com o id da PRIMEIRA
linha; com a ordem por tempo, a primeira linha deixa de ser a de maior id, e a
comparação passa a usar o maior id da lista.

## O que mais vale lembrar

- Paginação por cursor tem que usar o mesmo par da ordenação (`criada_em, id`);
  cursor só por id pula ou repete linhas quando a ordem não é a do id.
- A semente é o melhor teste disso: dado gerado fora de ordem denuncia a tela
  que confia no id.

## Conexões
- Princípio: [[Produtividade se mede pela hora do registro, não pela data do fato]] —
  toda linha tem as duas datas, e a pergunta da tela escolhe qual; o id é mais uma
  forma da data do registro.
- Irmã: [[Consulta sem ordem não é determinística, e semente que usa o índice muda sozinha]]
- Visto em: [[telebot]]
- Mapa: [[Dados]]
