---
tags: [tipo/atomica, camada/padrao, dev/backend, dados]
criado: 2026-08-12
---

# Campo que vira indicador é coluna, o resto do documento é jsonb

> Ao guardar um documento de forma fixa (um formulário, um laudo), decida a casa
> de cada campo pelo **uso**: o que você vai **agregar, filtrar ou ordenar** vira
> COLUNA; o texto livre e as listas de forma variável vão em **jsonb**.

## O problema

Um formulário longo tempta os dois extremos. Uma coluna por campo incha a tabela
com dezenas de textos que ninguém consulta e vira migração a cada ajuste do
formulário. Um jsonb só ("guarda o form inteiro num campo") deixa barato salvar,
mas todo indicador precisa cavar dentro do json — filtrar por criticidade,
somar por grupo, ordenar por data fica lento e feio.

## A solução

Corte pelo que a consulta precisa:

- **Coluna** para o que vira número na tela: identidade e situação (nº, status,
  autor), e as dimensões de indicador/filtro (criticidade, grupo, datas). Tipado,
  indexável, `where`/`group by` direto.
- **jsonb** para o que só se lê ou se manda pra IA: a narrativa (o que aconteceu,
  causa raiz, lições) e as **tabelas repetíveis** de forma variável (linha do
  tempo, 5 porquês, ações corretivas/preventivas). Muda o formulário? Muda o
  shape do jsonb, sem migração na tabela.

Regra prática: se aparece num `where`, `group by` ou `order by`, é coluna; se só
volta pra renderizar ou pra um laudo, é jsonb.

## O que mais vale lembrar

- No `node-pg`, parâmetro jsonb vai como **texto** (`JSON.stringify`) — objeto/array
  cru vira `"[object Object]"`. Na volta, o driver já entrega parseado.
- É diferente de [[Entidade núcleo cresce por tabela satélite, não por coluna]]:
  aquilo é sobre onde mora um atributo entre TABELAS (crescimento da entidade);
  isto é sobre a casa de um campo DENTRO de um registro (coluna x jsonb).

## Conexões
- Irmã: [[Entidade núcleo cresce por tabela satélite, não por coluna]] · [[Formulário montado pelo usuário — a definição no banco dirige renderer e validação]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Dados]]
