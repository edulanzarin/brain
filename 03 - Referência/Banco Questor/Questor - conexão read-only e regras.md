---
tags: [tipo/atomica, camada/referencia, dev/backend, banco/questor]
criado: 2026-07-18
---

# Questor - conexão read-only e regras

> O banco do Questor é produção. Toda conexão força read-only; o próprio Postgres recusa escrita.

## Conexão

- **Host** 192.168.5.225 · **Porta** 5432 · **Banco** Navecon · **Engine** PostgreSQL · **Usuário** postgres.
- Senha só em `.env.local` de cada projeto — nunca no código nem em nota versionada.

## Somente leitura (obrigatório)

```js
new Pool({
  host: "192.168.5.225", port: 5432, database: "Navecon", user: "postgres", password: process.env.SENHA,
  options: "-c default_transaction_read_only=on -c statement_timeout=60000",
});
```

Com `default_transaction_read_only=on` o Postgres recusa qualquer escrita
(`cannot execute INSERT/UPDATE/CREATE ... in a read-only transaction`). Verificado.
Nunca desligar essa flag.

## Performance

- Tabelas de nota são enormes: `lctofissai` ~18M, `lctofissaiproduto` ~47M, `lctofisent` ~7,4M, `lctofisentproduto` ~12M.
- Índice de ouro pra período: `(codigoempresa, codigoestab, datalctofis)` — em quase todas as `lctofis*`.
- **Sempre** filtrar `datalctofis between` (usa o índice); filtrar empresa reduz drástico.
- Agregar um mês de todas as empresas: cabeçalho ~0,5–0,9s; itens ~2,6s.
- `statement_timeout` (60s) evita que uma consulta ruim trave a conexão.

## Conexões
- Ver também: [[Modelo de dados fiscais do Questor]] · [[Receitas SQL do Questor]]
- Padrão de ranking sobre tabela gigante: [[Agregar antes de juntar em tabelas gigantes no Postgres]].
- Visto em: [[Questor Hub]]
- Índice do banco: [[Banco Questor]]
- Mapa: [[Banco Questor]]
