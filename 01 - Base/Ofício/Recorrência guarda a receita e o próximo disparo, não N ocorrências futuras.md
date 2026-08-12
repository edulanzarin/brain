---
tags: [tipo/atomica, camada/principio, dev/backend]
criado: 2026-08-12
---

# Recorrência guarda a receita e o próximo disparo, não N ocorrências futuras

> Uma coisa que se repete no tempo se modela como a receita mais um ponteiro de "próxima vez" — o job gera UMA ocorrência concreta quando a data chega e avança o ponteiro. Nunca se pré-cria a fila inteira de ocorrências futuras.

## A regra

Ao guardar algo periódico (um envio mensal, uma avaliação por marco, um lembrete),
guarde dois pedaços:

1. **a receita** — o que disparar e para quem (resolvido no momento do disparo);
2. **o próximo disparo** — uma data.

Quando o próximo disparo vence, o job **materializa uma ocorrência** (a linha
concreta daquele ciclo) e reprograma o ponteiro para a próxima data. O passado fica
registrado nas ocorrências já materializadas; o futuro é sempre uma data só.

## Por que

Pré-criar N linhas futuras apodrece: o público muda (alguém entra/sai do setor),
a data muda, a regra é editada — e você fica com uma fila de ocorrências obsoletas
para reconciliar. Resolver o alvo **na hora** do disparo dá sempre o estado de hoje
de graça. E um ponteiro único é idempotente: rodar o job duas vezes não duplica,
porque a segunda passada já vê o ponteiro no futuro.

Dois casos no mesmo sistema (Nexo) me convenceram de que é princípio, não detalhe:

- **Experiência**: o contrato do Questor projeta os marcos 45/90, mas a linha
  `rh_experiencia` só nasce quando o marco entra na janela — materializada sob
  demanda, não pré-criada para todo mundo.
- **Envio automático**: uma `envio_regra` guarda formulário + público + frequência +
  `proximo_disparo`; o cron resolve os destinatários hoje, cria uma campanha (`envio`)
  e avança `proximo_disparo`.

## Na prática

- O ponteiro no passado é o gatilho: `where ativo and proximo_disparo <= now()`.
- Reprograme **sempre** depois de tentar disparar — mesmo sem alvo ou com erro —,
  senão o job repete a regra a cada rodada.
- O alvo é uma consulta no disparo, não uma coluna congelada.

## Conexões
- Irmã: [[A definição em dado dirige o comportamento, não um caso no código]]
- Irmã: [[Um invariante se garante na estrutura, não no processo]]
- Técnica que aplica: [[Regra de envio recorrente materializa uma campanha e reprograma]]
- Mapa: [[Base]]
