---
tags: [tipo/atomica, camada/padrao, sql, dev/backend, armadilha]
criado: 2026-09-25
---

# Parâmetro posicional não se renumera quando a coluna sai

> Com `$1, $2, ...` escritos à mão, a lista de colunas, a lista de marcadores e o
> array de valores são três listas que só batem por disciplina. Tirar uma coluna
> de uma delas não quebra a compilação nem o lint: quebra no banco, na primeira
> vez que alguém usa o caminho, que pode ser semanas depois.

## O caso

No RH do Nexo (nexo2), a regra de envio automático perdeu a coluna `escopo`
quando o desempenho virou seção própria (28/08/2026). O commit tirou a coluna do
`insert` e o valor do array, mas não renumerou os marcadores:

```sql
insert into envio_regra (formulario_id, ..., alvo, freq_tipo, ...)  -- 11 colunas
values ($1, $2, $3, $4, $5, $6, $7::jsonb, $8, $9, $10, $11, $12)  -- 12 marcadores
```

E no `update` o `$6` sumiu e apareceu um `$12` que o array não tinha. Criar ou
editar regra automática falhava no banco. A lista de regras abria vazia, que é
exatamente o que uma lista sem regras mostra, e ninguém criou regra nova por um
mês. Quem achou foi o porte para o NaveX, ao montar a tela nova de novo.

## Como evitar

- **Monte os marcadores a partir das colunas**, numa estrutura só:
  `const campos = { formulario_id: x, titulo: y, ... }` e gere a lista de colunas,
  os `$n` e o array dela. Coluna que sai, sai das três de uma vez.
- Se ficar à mão, **conte na revisão**: colunas, marcadores e valores, os três
  números lado a lado. É o único teste que o diff de uma remoção precisa.
- Cast no marcador (`$7::jsonb`) é o que mais esconde o erro: o número do cast
  continua lá, apontando para o valor vizinho, e o erro do banco fala de tipo,
  não de posição.
- **Todo caminho de escrita tem um teste que escreve.** A tela de leitura não
  prova nada sobre ele; ver [[Recurso sem escrita parece pronto quando a semente preenche a leitura]].

## Conexões
- Princípio: [[Recurso sem escrita parece pronto quando a semente preenche a leitura]]
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Visto em: [[NaveX]] · [[Navetech Hub]]
- Mapa: [[Dados]]
