---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-18
---

# Falha de automação recorrente vira alerta com throttle, não catch vazio

> Operação que roda sozinha em loop (venda, compra, sync) não pode engolir erro em
> `catch {}`: a falha vira um evento visível ao usuário, com throttle pra não inundar.

## O problema

O robô vendia e comprava em timers com `catch { /* proxima tenta */ }` em volta de tudo.
Racional aparente: "falha pontual, a próxima varredura resolve". Mas quando a causa era
estrutural (token apodrecido, endpoint mudado), TODA varredura falhava igual — por
semanas, sem nenhum sinal. O usuário só via o efeito indireto: "não vende", "não
compra", "a fila trava". Debug às cegas, confiança no produto destruída.

## A solução

Um logger de falha operacional com throttle por operação:

```ts
private lastOpErrAt = new Map<string, number>();
private logOpError(op: string, title: string, body: string | null) {
  const last = this.lastOpErrAt.get(op) ?? 0;
  if (Date.now() - last < 30 * 60_000) return;   // no maximo 1 por operacao a cada 30min
  this.lastOpErrAt.set(op, Date.now());
  void logRobotEvent(userId, { kind: "error", title, body });
}
```

- Cada ponto de falha nomeia a operação (`sell-pokes`, `autobuy`) e diz o MOTIVO em
  linguagem de usuário ("o jogo recusou o token", "a loja não respondeu").
- O evento cai no mesmo feed de atividade que os sucessos — falha é atividade.
- O `catch` continua segurando a exceção (loop não pode morrer); a diferença é que agora
  ele REGISTRA antes de seguir.

## O que mais vale lembrar

- Throttle por operação, não global: venda falhando não silencia o alerta da compra.
- Falha transitória de verdade não aparece (a próxima varredura passa e o throttle nem
  dispara de novo); a estrutural aparece a cada 30min até ser corrigida — exatamente o
  sinal certo.

## Conexões
- Princípio: [[Chamada externa tem timeout e erro tratado]]
- Irmã: [[Token que rotaciona não tolera cópia longeva, releia do banco antes do uso]]
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
