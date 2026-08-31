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

### O `loggeral` está DESLIGADO nesta base (verificado ago/2026)

Parecia a mina de ouro para medir trabalho: tem módulo, operação, **duração**
(`datahorafim - datahorainicio`), resultado, estação — e, ao contrário de
`lctofis`/`lctoctb`, **tem índice em `datahorainicio`**, o que tornaria o
recorte por período barato.

Não serve, porque parou de ser escrito: **330 operações em jun/2026, 10 em
julho, ZERO em agosto**, com um único usuário e um único módulo. Os 11,5 milhões
de linhas são histórico acumulado até algum ponto do passado. Antes de desenhar
qualquer coisa em cima dele, confira o volume do mês corrente.

Dois detalhes de higiene do dado, se um dia voltar: há 3 linhas com
`datahorainicio` no FUTURO (a máxima é 2049) e 23 mil antes de 2015, então
`max()`/`min()` não servem para descobrir o alcance real.

## Auditoria embutida (o caminho barato)

Quase toda tabela transacional já carrega, por linha:

- `codigousuario` → `usuario` (`nomeusuario`). **`0` = ADMINISTRADOR** (conta do sistema; importações automáticas/e-Doc caem nele).
- `datahoralcto` / `datahoralctofis` / `datahoralctoctb` — carimbo de quando foi lançado.
- `origemdado` — como entrou (`3` integração, `2` importado, `1` manual).

Para "quem lançou esta nota / este lançamento contábil", leia essas colunas direto na tabela do dado — sem tocar em `loggeral`. (Foi assim que o [[Navetech Hub]] fez a análise de quem lançou a nota.)

## Rastros de exclusão/retificação

- `lctoctbexcluido` (~10,7M) — lançamentos contábeis **excluídos**.
- `lctofis*retif` / tabelas de retificação — histórico de retificações fiscais.

Úteis para auditar o que foi apagado/alterado sem sumir do banco.

### `lctoctbexcluido` é a linha inteira mais duas colunas (verificado ago/2026)

O Questor não deleta: **copia a linha completa** de `lctoctb` (todas as 27 colunas,
inclusive `codigousuario` e `datahoralctoctb` originais) e acrescenta `idlctoctbexcl`
(PK própria), **`dataexclusao`** (date, sem hora) e **`usuarioexclusao`** (smallint).

Isso responde as duas perguntas de uma vez: **quem apagou** e **de quem era**. E a
diferença `dataexclusao - datahoralctoctb::date` dá a **idade** do lançamento no dia em
que morreu — apagar no mesmo dia é conserto, apagar mês fechado de outra pessoa é outra
conversa.

- Volume real do escritório: 100 a 400 mil exclusões por mês, ~28 pessoas. Em ago/2026,
  43% do que cada um apagou tinha sido lançado por outra pessoa. Em fev/2026 houve pico
  de 1,68M (reimportação em massa) — exclusão em lote é rotina, não incidente.
- **Não há índice em `dataexclusao`** (só em `datalctoctb`), então filtrar por período de
  exclusão varre a tabela: ~1,3 s para um mês do escritório inteiro. Barato o bastante.
- `dataexclusao` é DATE: `between $1 and $2` já pega o dia inteiro, sem o `+ 1 dia` que o
  `datahoralctoctb` (timestamp) exige.

## `tempouso` — a única fonte que mede ESFORÇO em hora

Todo o resto do banco conta linhas produzidas. `tempouso` (~700 mil linhas, ~30 mil por
mês) conta **tempo**: uma linha por `(datauso, codigoempresa, codigousuario,
codigoatividade)` com `tempouso` em **segundos**.

É o que permite perguntar quanto de atenção cada cliente custa, em vez de só quantos
lançamentos gerou. Uma semana típica dá 25 a 40 h por pessoa — bate com jornada real.

Três limites que a leitura precisa respeitar:

- É do **Questor inteiro**, não do módulo: a mesma linha conta a hora de quem estava na
  folha. Para recortar um setor, cruze com quem produziu no fato daquele setor no
  período — e diga na tela quem ficou de fora.
- **Não tem `codigoestab`** — filtro de filial não morde aqui.
- `codigoatividade` existe e é inútil nesta base: só `0` e `9999` ("Não definido", da
  tabela `atividade`, que tem duas linhas). Não dá para quebrar por atividade.
- `horainicio`/`horafim`/`tiporegistro`/`descricao` vêm nulos; `usuario.custohora` está
  preenchido em 41 de 437 usuários, então **não** dá para converter hora em dinheiro.

## Por que importa

Distingue os dois níveis: auditoria leve (colunas na própria linha) para relatórios do dia a dia; log central para investigação forense. E evita o erro de varrer `logparam` (140M) sem necessidade.

## Conexões
- Índice do banco: [[Banco Questor]] · Convenções: [[Panorama e convenções do banco Questor]]
- Quem lançou a nota: [[Modelo de dados fiscais do Questor]]
- Mapa: [[Banco Questor]]
