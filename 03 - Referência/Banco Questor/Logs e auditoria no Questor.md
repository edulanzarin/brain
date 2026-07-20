---
tags: [tipo/atomica, camada/referencia, dev/backend, banco/questor]
criado: 2026-07-18
---

# Logs e auditoria no Questor

> Há dois níveis de auditoria: um **log central** de operações (`loggeral`/`logparam` — a maior tabela do banco, 140M linhas) e as **colunas de auditoria embutidas** em quase toda tabela transacional (`codigousuario`, `datahora*`, `origemdado`). Para "quem fez o quê" numa nota/lançamento, use as colunas — é muito mais barato.

## Log central de operações

- **`loggeral`** (~11,5M) — uma linha por operação: `idlog` PK, `datahorainicio`/`datahorafim`, `resultado`, `codigousuario` → `usuario`, `codigomodulo` → `permodulo`, `codigologoperacao` → `logoperacao`, `codigologdescr` → `logdescr` (texto), `codigologestacao` → `logestacao` (máquina/estação), `codigologobjeto` → `logobjeto`.
- **`logparam`** (~140M — **maior tabela do banco**) + `logparamvalor` (~4,3M) — os parâmetros/valores de cada operação logada (`idlog`, `codigoparam`, `pos`). Auditoria fina; **pesadíssima, evitar varrer** — só acessar por `idlog` específico.
- `logdescr` (~1,5M) — textos das descrições de log.

Serve para trilha de auditoria detalhada (quem rodou qual rotina, quando, de onde). Caro; não é para relatório de rotina.

## Auditoria embutida (o caminho barato)

Quase toda tabela transacional já carrega, por linha:

- `codigousuario` → `usuario` (`nomeusuario`). **`0` = ADMINISTRADOR** (conta do sistema; importações automáticas/e-Doc caem nele).
- `datahoralcto` / `datahoralctofis` / `datahoralctoctb` — carimbo de quando foi lançado.
- `origemdado` — como entrou (`3` integração, `2` importado, `1` manual).

Para "quem lançou esta nota / este lançamento contábil", leia essas colunas direto na tabela do dado — sem tocar em `loggeral`. (Foi assim que o [[Questor Hub]] fez a análise de quem lançou a nota.)

## Rastros de exclusão/retificação

- `lctoctbexcluido` (~10,5M) — lançamentos contábeis **excluídos**.
- `lctofis*retif` / tabelas de retificação — histórico de retificações fiscais.

Úteis para auditar o que foi apagado/alterado sem sumir do banco.

## Por que importa

Distingue os dois níveis: auditoria leve (colunas na própria linha) para relatórios do dia a dia; log central para investigação forense. E evita o erro de varrer `logparam` (140M) sem necessidade.

## Conexões
- Índice do banco: [[Banco Questor]] · Convenções: [[Panorama e convenções do banco Questor]]
- Quem lançou a nota: [[Modelo de dados fiscais do Questor]]
- Mapa: [[Banco Questor]]
