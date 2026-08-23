---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-24
---

# Comando sem resposta precisa de vigia, não de fé

> Protocolo de tempo real costuma ter comandos que não respondem nada: você
> manda o frame e o servidor aceita ou ignora. Escrever isso é uma linha, e a
> linha parece completa. Ela não é: **um comando sem resposta que se perde não
> produz erro, produz silêncio** — e silêncio se parece com "está funcionando,
> só não aconteceu nada ainda".

## O caso

No [[piwdex2]] o robô entra na caçada mandando `enter-hunt {slug}` pelo
WebSocket. Sem ack, sem retorno. Em operação normal funciona sempre.

Depois de um deploy, não. A conexão nasce enquanto o servidor do jogo ainda está
trocando de estado, o frame se perde, e o que sobra é o pior estado possível:
socket aberto, sessão válida, analyzer zerado, e a tela dizendo "a conexão está
aberta e o jogo não está mandando combate" — indefinidamente. Nada falhou. O robô
segurava a conta do dono a noite inteira sem caçar.

## O vigia

A correção não é reenviar mais; é **observar o efeito e reagir à ausência dele**.
O comando tinha uma consequência observável (o stream de combate), e era ela que
ninguém estava olhando.

```ts
// de carona no poll que já existe (a cada 2s)
private conferirCampo() {
  if (!this.ws || !this.slug) return;
  if (this.caidoDesde != null || this.deveVoltarAoCampo) return; // mudez COM dono
  const agora = Date.now();
  if (agora - this.ultimoCampoEm  < TRAVADO_MS) return; // ainda chega frame
  if (agora - this.reentradaEm    < TRAVADO_MS) return; // já reenviei há pouco
  if (this.desdeMs && agora - this.desdeMs < TRAVADO_MS) return; // recém-conectado
  this.reentradaEm = agora;
  this.enviar({ type: "enter-hunt", slug: this.slug });
}
```

- **Reenviar tem que ser idempotente.** Entrar num campo em que já se está não
  faz nada; é isso que torna o vigia barato. Se o comando não for idempotente, o
  vigia precisa confirmar antes de repetir.
- **Excluir a mudez que tem dono.** Líder desmaiado também deixa o campo mudo, e
  ali quem resolve é outra rotina. Vigia que ignora isso briga com ela.
- **Dar prazo à conexão nova.** O primeiro frame demora; cobrar antes disso faz
  o vigia disparar em toda conexão saudável.
- **Registrar quando agir.** "Reentrei porque o campo estava mudo" é a linha que
  transforma um sintoma vago em causa na próxima vez.

## O que mais vale lembrar

A pergunta que revela esses casos: **"que evento me avisa se este comando NÃO
tiver efeito?"** Se a resposta for "nenhum", falta o vigia. É a mesma pergunta
que descobre o socket que nunca abre — a diferença é que lá o buraco está na
conexão, e aqui no comando que passa por ela.

## Conexões
- Princípio: [[Laço que trata toda falha igual apaga a causa da primeira]]
- Irmã: [[Socket que não abre não emite evento, e só um temporizador percebe]] ·
  [[Confirme a mutação pelo estado que ela deixa, não pelo ack que pode não chegar]] ·
  [[Comando que responde ok e não muda nada tem pré-condição de estado]]
- Visto em: [[piwdex2]]
- Mapa: [[Backend]]
