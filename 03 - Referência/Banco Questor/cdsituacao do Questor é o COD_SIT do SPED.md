---
tags: [tipo/atomica, camada/referencia, dev/backend, banco/questor]
criado: 2026-09-25
---

# cdsituacao do Questor é o COD_SIT do SPED

> `cdsituacao` em `lctofisent`/`lctofissai` segue a tabela de situação do documento
> do SPED Fiscal (campo COD_SIT do registro C100). Conferido pelo comportamento do
> dado em set/2026, não por documentação: o banco não tem cadastro de nome para ele.

## Os códigos

| código | situação | é problema? |
|---|---|---|
| 0 | Regular | não |
| 1 | Extemporânea (documento regular escriturado fora do prazo) | atenção |
| 2 | Cancelada | é a cancelada (junto com `cancelada = '1'`) |
| 3 | Cancelada extemporânea | idem |
| 4 | Denegada | sim |
| 5 | Inutilizada (numeração que nunca virou nota) | sim |
| 6 | Complementar | não, é documento regular com valor |
| 7 | Complementar extemporânea | atenção |
| 8 | Regime especial | não |

## A prova (saídas de jan a ago/2026)

- **4**: 4 notas, todas NF-e, todas com valor zero.
- **5**: 2.144 notas, **todas com valor zero e sem chave de acesso**, 1.633 delas
  NFC-e. Inutilizar faixa de numeração é rotina no varejo.
- **6**: 5.810 notas, valor médio de R$ 388 e 85% com valor, 3.441 CT-e e 2.369
  NF-e. CT-e complementar (frete complementar) é comum; CT-e denegado nessa
  quantidade não seria.
- Nas entradas aparecem ainda o **1** (2.374 NF-e) e o **8** (6.733 NF-e).

## Consequências para consulta

- "Denegadas ou inutilizadas" é `cdsituacao in (4, 5)`. Contar `cdsituacao <> 0`
  põe as complementares (6) e o regime especial (8) na conta.
- "Nota sem chave de acesso" precisa excluir o 5: inutilizada não tem chave por
  definição. Sem a exclusão, 2.144 das 2.150 "sem chave" do período eram
  inutilizadas; a pendência real eram 6.
- O nexo2 ([[Navetech Hub]]) lia 5 como denegada, 6 como inutilizada e contava
  tudo que não era 0. O [[NaveX]] nasceu corrigido.

## Conexões
- Princípio: [[Código sem cadastro se prova pelo comportamento do dado, não pelo rótulo herdado]]
- Irmã: [[Canceladas e devoluções no Questor]] · [[Modelo de dados fiscais do Questor]]
- Visto em: [[NaveX]]
- Mapa: [[Banco Questor]]
