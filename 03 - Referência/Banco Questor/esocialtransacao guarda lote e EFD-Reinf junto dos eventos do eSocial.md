---
tags: [tipo/atomica, camada/referencia, dev/backend, banco/questor]
criado: 2026-09-25
---

# esocialtransacao guarda lote e EFD-Reinf junto dos eventos do eSocial

> `esocialtransacao` não é só evento do eSocial. Tem o envelope de cada lote
> enviado (`ENVIOLOTEEVENTOS`, `ENVIOLOTEEVENTOSEFDREINF`), que nunca tem recibo
> porque o recibo é de cada evento, e os eventos da EFD-Reinf (`R-xxxx`), que são
> obrigação fiscal. Contar eSocial é `evento like 'S-%'`.

## O tamanho

Últimos 90 dias até set/2026, escritório inteiro, transmissões sem recibo e sem
rejeição (`coalesce(status, 0) <> 13`):

- 39.615 contando a tabela inteira;
- 20.189 eram `ENVIOLOTEEVENTOS` e 3.144 `ENVIOLOTEEVENTOSEFDREINF`;
- com só `S-%`, sobram 15.967. Rejeitadas (status 13) caem de 3.332 para 2.902.

## Status observados

Sem cadastro de nome para `status`. O que o dado mostra:

| status | recibo | leitura |
|---|---|---|
| 8 | sim | o aceito mais comum (56 mil em 90 dias) |
| 8 | não | quase todo envelope de lote, sem `track` |
| 2 | não | 13,9 mil, sem leitura provada (a checagem por track não serve, ver abaixo) |
| 13 | não | rejeitado (ver [[Módulo de folha e eSocial do Questor]]) |
| 15, 9 | sim | também aceitos, espalhados por vários tipos de evento |

A regra de situação continua: recibo preenchido = aceito; sem recibo e status
13 = rejeitado; o resto = pendente.

## Retransmissão não se deduplica pelo track

O `track` de evento periódico é da agenda, não do evento: todos os S-1200 de uma
empresa num envio saem com `(AGENDA1200::1)`. Agrupar por empresa, evento e
track junta funcionários diferentes. Para os eventos de contrato, a ligação
certa é a `esocialdadoss<NNNN>` (a última transmissão por `datahoralcto`). Para
os periódicos, a chave do evento ainda está por achar: por isso o panorama
conta transmissões, e uma tentativa refeita conta de novo.

## Conexões
- Princípio: [[Código sem cadastro se prova pelo comportamento do dado, não pelo rótulo herdado]]
- Irmã: [[Módulo de folha e eSocial do Questor]]
- Visto em: [[NaveX]] · [[Navetech Hub]]
- Mapa: [[Banco Questor]]
