---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-19
---

# Recusa não é falha: contra o não do servidor, insistir é ruído

> Retry existe pra falha **passageira** — rede caiu, servidor engasgou, token venceu.
> Quando o outro lado **recusa** (banido, sem permissão, cota estourada), tentar de novo
> não muda o resultado: só gasta tentativa contra quem já respondeu não, e esconde do
> usuário o motivo real. Classifique a resposta antes de decidir se vale insistir.

## O problema

Camada de integração costuma ter um caminho de erro só: falhou, agenda o retry. Isso
funciona pro caso comum e falha exatamente no caso que mais importa, porque **as duas
situações chegam com a mesma cara** — uma exceção, ou um socket que fechou.

No [[piwdex]] uma conta foi banida pelo jogo. O robô abria o WebSocket, o jogo fechava,
o motor lia "caiu" e reconectava — para sempre, com backoff, a cada restart do container.
O usuário via só "não conectou", reconectava na mão, e nunca descobria que o jogo tinha
recusado a conta. O sistema **martelava a porta de quem já tinha dito não** e ainda
guardava o motivo pra si.

O agravante parecia ser que **socket não conta o motivo**. Não era: em ago/2026 a
sondagem do mesmo jogo mostrou que ele fecha com `4001 unauthorized` e `4003
wrong-shard` — a faixa 4000–4999 do WebSocket existe pra aplicação dizer o que
houve, e o motor estava descartando a resposta. Antes de concluir que o
transporte é mudo, leia o código e o `reason`: ver
[[O código com que o socket fecha é a classificação que o retry precisa]].

## A solução

Três decisões separadas: **classificar**, **decidir o retry**, **contar ao humano**.

```ts
export type RefusalKind =
  | "blocked"       // 403: recusado. TERMINAL — não reconectar.
  | "expired"       // 401 depois do refresh: o vínculo morreu, reconectar resolve.
  | "rate_limited"; // 429: pediu pra desacelerar. Esperar, não desistir.

export async function refusalOf(res: Response): Promise<Refusal | null> {
  if (res.ok) return null;
  if (res.status === 403) return { kind: "blocked", status: 403, message: await body(res) };
  if (res.status === 401) return { kind: "expired", status: 401, message: await body(res) };
  if (res.status === 429) return { kind: "rate_limited", status: 429, message: await body(res) };
  return null; // 5xx: problema do servidor, não recusa — retry normal
}
```

- **Quando o canal não sabe recusar, pergunte a um que saiba.** Antes de reabrir o
  socket, uma chamada REST responde com código HTTP. Uma pergunta barata substitui um
  loop de tentativas que nunca ia entender a resposta. Vale como rede de proteção para
  o fechamento genérico (`1006`), depois de confirmar que o transporte realmente não
  classificou.
- **Recusa terminal desliga a intenção**, não só a conexão do momento. Se o "quero
  rodando" continua ligado no banco, o próximo boot religa e o loop recomeça — ver
  [[Estado desejado persistido religa o robô depois do restart]], que é a mesma
  propriedade agindo contra você aqui.
- **Guarde a resposta crua do outro lado**, truncada. A frase do ban é de quem baniu:
  mostrá-la é a única prova de que o problema não é seu sistema. De quebra, a detecção
  não depende de casar texto — se ele reescrever a mensagem, o código continua valendo.

## O que mais vale lembrar

- **Não invente a regra antes da evidência.** Se você não sabe qual código o serviço usa
  pra recusar, faça a falha não classificada mostrar o **status e o corpo**. A regra se
  ajusta com o caso real na mão, em vez de nascer de um chute que ninguém revisita.
- Recusa é estado **terminal com saída**: sai quando a credencial nova é aceita. Limpe a
  marca no sucesso, senão o usuário resolve com o suporte e o sistema segue dizendo não.
- `429` parece recusa e não é: ali insistir é certo — só que devagar.

## Conexões
- Princípio: [[Chamada externa tem timeout e erro tratado]]
- Irmã: [[O código com que o socket fecha é a classificação que o retry precisa]] ·
  [[Falha de automação recorrente vira alerta com throttle, não catch vazio]] (o
  outro lado da moeda: lá o erro estrutural era engolido, aqui era confundido com
  transitório) · [[Estado desejado persistido religa o robô depois do restart]]
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
