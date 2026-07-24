---
tags: [tipo/atomica, camada/referencia, dev/backend, banco/questor]
criado: 2026-07-18
---

# grupoprocessam do Questor não é grupo de empresas

> As tabelas `grupoprocessam` / `grupoempresa` do Questor são grupos de processamento interno, não agrupamentos de empresas para relatório.

## O quê

Ao montar o [[Navetech Hub]], a intuição foi usar `grupoprocessam` (tem `descrgrupoproc` com nomes tipo "Empresas Larissa") + `grupoempresa` (vínculo `codigogrupoproc`↔`codigoempresaproc`) como "grupos de empresas" pra filtrar. **Errado**: esses grupos existem pra rotinas internas do Questor (processamento em lote, cálculo de folha etc.), não pra análise. Os nomes enganam.

Solução adotada: grupos de empresas viram um recurso **do próprio app**, salvos no navegador (localStorage), onde o usuário monta o grupo escolhendo as empresas. Nada disso toca o banco do Questor.

## Por que importa

Evita basear filtros de BI numa dimensão que não significa o que parece. Vale a lição geral: nome de tabela/coluna em ERP legado não é contrato — confirmar o significado com o usuário/dados antes de construir em cima.

## Conexões
- Relacionado: [[Modelo de dados fiscais do Questor]] · [[Cadastros centrais do Questor - empresa, estab, pessoa]]
- Visto em: [[Navetech Hub]]
- Índice do banco: [[Banco Questor]]
- Mapa: [[Banco Questor]]
