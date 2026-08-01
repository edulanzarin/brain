---
tags: [tipo/atomica, camada/padrao, dev/backend]
criado: 2026-07-31
---

# Fila de exceção recomputa o achado e persiste só a triagem

> Uma fila de pendências sobre dado que você não possui (lê read-only) recomputa os achados a cada carga e guarda no seu banco só a DECISÃO humana sobre cada um — resolver/ignorar — ligada por uma identidade estável.

## O problema

Você tem telas que acham exceções numa fonte read-only (notas não contabilizadas, lançamentos anômalos) e quer transformá-las numa FILA de trabalho: marcar "já resolvi" / "pode ignorar" e não ver de novo. A tentação é gravar os achados numa tabela e pôr o status neles. Erro: o achado é derivado da fonte, que muda — a nota foi contabilizada depois, o lançamento corrigido. A cópia gravada apodrece, e você passa a triar fantasmas (itens que já não são problema) enquanto some o que virou problema agora.

## A solução

Separe o FATO da DECISÃO. O achado é **recomputado ao vivo da fonte a cada carga** (nunca persistido). No seu banco mora só a triagem: `(fonte, chave, tipo) → status, quem, quando`. Ao montar a fila, recompute os achados e faça `LEFT JOIN` da triagem por essa identidade.

A identidade tem que ser **estável e derivável do próprio achado**, não um id gerado: `fonte` + `chave` do registro na origem + `tipo` do problema. Um `unique (fonte, chave, tipo)` faz a triagem colar idempotente e sobreviver ao recompute ([[Um invariante se garante na estrutura, não no processo]]).

Pôr o `tipo` na chave dá um comportamento certo de graça: se o mesmo registro muda de problema (era "não contabilizada", virou "conta errada"), é outra pendência — reaparece sozinha, porque a triagem antiga era de outro tipo.

```
-- só a decisão, não o achado
create table conf_triagem (
  fonte text, codigo_empresa int, chave text, tipo text,
  status text,                 -- 'resolvido' | 'ignorado'
  usuario_nome text, atualizado_em timestamptz,
  unique (fonte, codigo_empresa, chave, tipo)
);
-- upsert on conflict grava a decisão; reabrir = apagar a linha
-- (o achado volta a contar como aberto no próximo recompute)
```

## O que mais vale lembrar

- É primo do [[Fluxo de fechamento é orquestração dos motores que já existem]]: os achados vêm de motores já validados (a conferência, a auditoria); a fila só os une e anota, não recalcula regra nenhuma.
- A triagem é **anotação, não realimenta o cálculo** — ao contrário do override do [[De-para determinístico com override que vira aprendizado]], que VOLTA pro motor. Guardar só o que não muda o cálculo é o que mantém a fonte como verdade única.
- Grave a triagem no SEU banco, nunca na fonte read-only. Snapshot de quem/quando (o nome sobrevive à remoção do usuário).
- Bônus de posicionamento: quando existe um robô que ESCREVE na fonte (auto-lançamento), a sua fila read-only vira a auditoria independente dele — recomputar o achado é justamente o que pega o erro que o robô cometeu.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[Fluxo de fechamento é orquestração dos motores que já existem]]
- Irmã: [[De-para determinístico com override que vira aprendizado]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Backend]]
