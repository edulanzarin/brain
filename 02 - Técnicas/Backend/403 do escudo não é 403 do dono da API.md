---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-24
---

# 403 do escudo não é 403 do dono da API

> Toda API séria tem uma borda na frente — Cloudflare, WAF, rate limiter de CDN.
> Ela responde com os MESMOS status da aplicação, e um `403` dela não fala da sua
> conta: fala do seu pedido. Tratar os dois como um só transforma um bloqueio de
> trinta segundos num estado terminal.

## O caso

No [[piwdex2]] o robô lia `403` como "o jogo recusou esta conta" e marcava o
vínculo `blocked` — que é terminal de propósito: reconectar não desfaz ban, e
insistir a cada restart era o comportamento que o desenho existia para eliminar.

Aí o `403` chegou com isto no corpo:

```html
<!DOCTYPE html><html lang="en-US"><head><title>Just a moment...</title>
<meta name="robots" content="noindex,nofollow">
```

É o desafio do Cloudflare. A conta estava inteira. E a tela dizia, com todas as
letras, *"o jogo recusou esta conta — reconectar não resolve"* — a frase mais
desanimadora possível para um problema que a próxima tentativa resolveria.

## Como separar

A assinatura é o **corpo**, não o status:

- **API responde JSON.** Recusa de conta vem com `message`/`error`.
- **Escudo responde HTML.** `<!doctype`, `<title>`, `noindex,nofollow`, CSP.
- **Header quando vem**: `cf-mitigated`, `cf-ray`, `server: cloudflare`.

```ts
export function ehEscudo(res: Response, corpo: string): boolean {
  if (res.headers.get("cf-mitigated")) return true;
  const html = (res.headers.get("content-type") ?? "").includes("text/html")
    || corpo.trimStart().toLowerCase().startsWith("<!doctype");
  return html && /just a moment|cf_chl|attention required|cloudflare/i.test(corpo);
}
```

E o tratamento é oposto: escudo **não grava nada** no vínculo e cai no backoff
normal; recusa de conta grava e para.

## O que reduz o desafio

O escudo julga o pedido inteiro, não só o `User-Agent`. Um GET com `Bearer` e
mais nada não se parece com o que um navegador manda. Mandar o que o cliente
oficial manda — `Accept`, `Accept-Language`, `Origin`, `Referer`, `Sec-Fetch-*` —
não é disfarce: é fazer o mesmo pedido que a aba faria.

## O que mais vale lembrar

- **Estado terminal exige evidência forte.** Antes de gravar "não tente mais",
  pergunte se a resposta veio mesmo de quem você acha. O custo do falso positivo
  aqui é assimétrico: um retry a mais custa uma requisição, um `blocked` errado
  custa a conta do usuário até alguém perceber.
- Vale para `429` (do CDN x do app), `503` (manutenção x borda), e para qualquer
  status lido de uma API que você não hospeda.

## Conexões
- Princípio: [[Laço que trata toda falha igual apaga a causa da primeira]]
- Irmã: [[O código com que o socket fecha é a classificação que o retry precisa]] ·
  [[Recusa não é falha: contra o não do servidor, insistir é ruído]] ·
  [[Chamada externa tem timeout e erro tratado]]
- Visto em: [[piwdex2]]
- Mapa: [[Backend]]
