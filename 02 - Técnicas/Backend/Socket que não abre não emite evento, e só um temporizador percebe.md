---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-23
---

# Socket que não abre não emite evento, e só um temporizador percebe

> Toda a lógica de reconexão pendura no `close` e no `error`. Os dois pressupõem
> que a conexão chegou a existir. Quando o handshake simplesmente não completa —
> a rota some no meio, o servidor aceita o TCP e nunca responde o upgrade — não
> há `open`, não há `close`, não há `error`. Formalmente nada falhou, então o
> agendador de reconexão nunca é chamado.

## Como isso aparece

Não aparece como erro. Aparece como um status que não muda: `conectando` há 6
segundos, depois há 40, depois há minutos. Nada no log, porque não houve exceção.
Foi o estado mais difícil de diagnosticar que o motor do [[piwdex]] produzia, e
ele não tinha sintoma nenhum além do relógio.

O agravante é que o estado é **plausível**: conectar leva mesmo alguns segundos, e
quem olha a tela não sabe quando "ainda conectando" virou "travado".

## A solução

Um teto de abertura, armado junto com o socket e desarmado no `open`. Ele é o
único participante que enxerga o não-evento.

```ts
this.tAbertura = setTimeout(() => {
  if (minhaGeracao !== this.geracao || this.status !== "conectando") return;
  registrarFalha(null, "o servidor não respondeu ao pedido de conexão");
  try { ws.close(); } catch {}      // pode nem ter aberto
  this.ws = null;
  if (this.ligado) this.agendarReconexao();
}, ABERTURA_MS);
```

- **A guarda de geração é obrigatória.** O temporizador sobrevive ao socket que o
  armou; sem checar se ele ainda é o atual, ele derruba a conexão seguinte, que
  já estava boa.
- **Feche o socket órfão.** Ele pode completar o handshake depois e ficar aberto
  sem ninguém escutando, disputando a sessão contra o substituto.
- **Registre uma falha com motivo próprio.** "Sem resposta" é diferente de "caiu",
  e essa diferença aponta rede/rota em vez de credencial.

## O que mais vale lembrar

- O mesmo buraco existe em qualquer recurso que abre por handshake: stream de
  banco, gRPC, SSE do lado do servidor. A pergunta é sempre "que evento me avisa
  se isto **não** acontecer?" — e a resposta honesta costuma ser "nenhum".
- Estado de espera na tela precisa de **prazo**, não só de rótulo. Um `conectando`
  que pode durar para sempre é indistinguível de um travamento.

## Conexões
- Princípio: [[Laço que trata toda falha igual apaga a causa da primeira]]
- Irmã: [[O código com que o socket fecha é a classificação que o retry precisa]] ·
  [[Chamada externa tem timeout e erro tratado]]
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
