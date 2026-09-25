---
tags: [tipo/atomica, camada/padrao, sql, dev/backend, armadilha]
criado: 2026-09-25
---

# Postgres de container nasce em UTC, e a hora formatada sem fuso mente

> A imagem oficial do Postgres sobe com `timezone = 'UTC'`. A coluna
> `timestamptz` guarda o instante certo, mas tudo que o banco escreve como texto
> ou recorta por calendário usa o fuso da SESSÃO: `to_char` sai em UTC, a hora
> tirada com `extract(hour)` sai em UTC, e `criado_em >= '2026-09-01'` corta à
> meia-noite de Greenwich. No Brasil, tudo 3 horas adiantado.

## O sintoma

Nada dá erro. A tela mostra "respondido às 14:32" para uma resposta das 11:32,
o mapa de horas da produtividade mostra o escritório trabalhando das 11h às 21h,
e o que acontece depois das 21h cai no dia (e às vezes no mês) seguinte.

A hora sai sem fuso quando a consulta faz
`to_char(criado_em, 'YYYY-MM-DD"T"HH24:MI:SS')` para entregar ao front: o
navegador lê o texto sem offset como hora local e acredita. O `Date` que o
driver `pg` devolve de um `timestamptz` não tem o problema (carrega o instante),
e por isso o defeito aparece só em parte das telas.

## A correção

Abrir a sessão no fuso do negócio, no pool:

```ts
new Pool({
  connectionString: process.env.APP_DB_URL,
  options: `-c statement_timeout=30000 -c timezone=America/Sao_Paulo`,
});
```

Uma linha conserta todas as consultas de uma vez: formatação, `extract`, corte
de mês com data literal, `current_date`, `date_trunc`. O fuso vem de variável
de ambiente com padrão, e o valor é validado antes de ir para a string de
opções (nome IANA, só letras, barra e sublinhado).

Alternativas piores:

- `at time zone` em cada consulta: 35 lugares, e o próximo esquece.
- `TZ` no container do banco: conserta este ambiente e não o próximo, e não vale
  para quem conecta de fora (script, migração).
- Formatar com offset (`OF`): o texto fica certo, mas `extract` e o corte de mês
  continuam em UTC.

## Onde não mexer

Banco de terceiro, lido e não gravado, tem o fuso dele (o Questor roda no fuso
local do servidor). A opção vai só no pool do banco do app.

## Conexões
- Princípio: [[Configuração vem do ambiente, não do código]]
- Irmã: [[Numeric e bigint do Postgres chegam como string no driver pg]]
  (outro padrão do driver e do servidor que ninguém escolheu e todo mundo herda)
- Irmã: [[Produtividade se mede pela hora do registro, não pela data do fato]]
  (a hora do registro só serve se estiver no fuso de quem trabalhou)
- Irmã: [[Agendador em container conta as horas em UTC]]
  (o processo Node ao lado nasce em UTC do mesmo jeito)
- Visto em: [[NaveX]] · [[Navetech Hub]]
- Mapa: [[Dados]]
