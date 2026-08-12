---
tags: [tipo/atomica, camada/padrao, dev/backend, sql]
criado: 2026-08-12
---

# Regra de envio recorrente materializa uma campanha e reprograma

> Uma regra de envio automático é uma linha (formulário + público + frequência + `proximo_disparo`); o cron pega as vencidas, resolve o público de hoje, cria UMA campanha reusando o disparo manual, e avança o ponteiro.

## O problema

O Nexo já tinha envio pontual de formulário (campanhas one-shot, agendáveis). Faltava
o **recorrente e robusto**: mandar o formulário de desempenho todo mês, escolhendo
setores ou colaboradores, e decidindo se quem responde é o gestor ou o colaborador.
Pré-agendar N campanhas futuras apodrece (ver o princípio).

## A solução

Uma tabela `envio_regra` guarda a receita e o ponteiro:

- **dois eixos** iguais aos do envio manual: `destinatario_tipo` (gestores |
  colaboradores) e, quando o gestor responde, `escopo` (generico | sobre_colaborador);
- **alvo flexível**: `alvo_tipo` (todos | setores | colaboradores) + `alvo` jsonb
  (`['CLASSIF', ...]` ou `[{empresa, contrato}, ...]`);
- **frequência**: `freq_tipo` (dias | mensal) + `freq_valor`;
- `ativo`, `ultimo_disparo`, `proximo_disparo`.

O job (`processarRegrasRecorrentes`, chamado dentro de `/api/rh/cron/envios`):

```
select ... where ativo and proximo_disparo <= now()
para cada regra:
  resolve o público HOJE (Diretório + gestores) -> params do criarEnvio existente
  criarEnvio(...)                    // reusa o disparo manual, não duplica lógica
  update proximo_disparo = próxima ocorrência   // SEMPRE, mesmo sem alvo/erro
```

O `proximoDisparo(freqTipo, freqValor, base)`: `dias` = base + N dias; `mensal` =
próxima ocorrência do dia do mês (limitado a 1–28 para não pular fevereiro).

## O que mais vale lembrar

- **Reaproveitar o caminho manual** (`criarEnvio`) foi o que manteve o motor pequeno:
  a regra só traduz seu público nos mesmos `destinatarios[]`/`colaboradores[]` que a
  tela já mandava.
- Resolver o alvo no disparo dá o estado de hoje de graça (quem entrou/saiu do setor).
- CTE `with novo as (insert ... returning *) select ... from novo join formulario`
  devolve a linha já com o nome do formulário numa ida só ao banco.
- O scheduler que bate a rota é peça separada:
  [[Agenda recorrente é um serviço do compose, não um crontab do host]].

## Conexões
- Princípio: [[Recorrência guarda a receita e o próximo disparo, não N ocorrências futuras]]
- Irmã: [[Uma resposta canônica de um grupo é um token compartilhado]]
- Depende de: [[Agenda recorrente é um serviço do compose, não um crontab do host]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Backend]]
