---
tags: [tipo/atomica, camada/padrao, dev/backend, sql, armadilha]
criado: 2026-09-17
---

# Regravar o conjunto de uma chave com delete e insert exige trava por chave

> "Apaga tudo da empresa e insere de novo" dentro de uma transação parece
> atômico, e sozinho é. Duas execuções ao mesmo tempo para a MESMA chave não
> são: a segunda morre em `duplicate key`. Quem serializa é uma trava consultiva
> transacional com a chave, tomada antes do delete.

## O problema

O padrão aparece em todo cadastro que se reaprende de um histórico: a função lê
o que o ERP fez, apaga as linhas da empresa e grava o resultado novo. Some o que
sumiu do histórico, entra o que apareceu, e a transação garante que ninguém vê o
meio do caminho.

O que a transação não garante é a ordem entre duas delas. Em `READ COMMITTED`:

- **Empresa ainda sem linhas.** As duas apagam o nada (delete de zero linhas não
  trava nada), as duas inserem as mesmas chaves. A segunda espera no índice
  único até a primeira confirmar, e aí falha.
- **Empresa com linhas.** A segunda espera no delete, pelas linhas que a primeira
  apagou. Quando a primeira confirma, o delete da segunda reavalia e não enxerga
  as linhas que a primeira acabou de inserir (não estavam no retrato). Insere por
  cima, e falha do mesmo jeito.

Não precisa de dois usuários para acontecer. Basta uma tela que dispare duas
consultas em paralelo e as duas acionarem o aprendizado da mesma empresa — foi
uma central de pendências rodando a conferência de entradas e a de saídas juntas,
numa empresa aberta pela primeira vez. A tela recebia 503.

## A saída

Tomar, logo depois do `begin`, uma trava consultiva de transação com a chave do
conjunto:

```sql
select pg_advisory_xact_lock($1, $2);  -- ($1 = namespace da função, $2 = empresa)
```

A segunda execução espera a primeira terminar e regrava o mesmo resultado.
Empresas diferentes não se esperam. A trava cai sozinha no commit ou no
rollback, então não há o que liberar à mão nem trava órfã se o processo morrer.

A forma de dois inteiros separa os usos: cada função que regrava um cadastro
tem a sua constante de namespace, e a trava de uma não segura a outra.

## Por que não outra coisa

- **`on conflict do nothing`** esconde o erro e mantém a corrida: a segunda
  pode ter lido o histórico antes de uma mudança e deixar linhas velhas
  misturadas às novas.
- **Upsert linha a linha** não apaga o que sumiu do histórico, que era o motivo
  de regravar o conjunto.
- **`serializable`** resolve devolvendo erro de serialização, e aí é preciso
  laço de nova tentativa em quem chama.

O índice único estava certo desde o começo: foi ele que transformou a corrida
em erro visível, em vez de linha duplicada calada.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[Consumir recurso de uso único é UPDATE condicional, não checar antes]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Dados]]
