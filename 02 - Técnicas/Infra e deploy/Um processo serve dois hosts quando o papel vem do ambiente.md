---
tags: [tipo/atomica, camada/padrao, infra]
criado: 2026-08-23
---

# Um processo serve dois hosts quando o papel vem do ambiente

> Quando duas metades de um sistema precisam de **cadências de deploy
> diferentes**, elas viram dois serviços — mas não precisam virar dois builds. A
> mesma imagem sobe duas vezes, e uma variável de ambiente decide qual host cada
> processo atende.

## Por que separar

O motivo não é organização. É
[[Processo que guarda conexão viva não tolera deploy frequente, e o log não denuncia]]:
a metade que segura WebSocket morre inteira a cada publicação, e a metade que
vive de SEO é publicada várias vezes por dia. Enquanto dividiam um serviço, cada
ajuste de texto derrubava a sessão de todo mundo.

**Cadência de deploy é propriedade do serviço, não do repositório.** O código
pode continuar junto — e é melhor que continue, senão o sistema de design e os
motores compartilhados viram duas cópias que divergem.

## Como

**Uma variável decide o papel**, e o padrão dela é o estado seguro:

```ts
export const PAPEL = (() => {
  const v = process.env.PIW_ROLE?.trim().toLowerCase();
  if (v === "site" || v === "bot" || v === "ambos") return v;
  // Ausente em produção vale o serviço que JÁ está no ar. Esquecer a variável
  // deixa o site intacto e a metade nova apagada.
  return process.env.NODE_ENV === "production" ? "site" : "ambos";
})();
```

Em desenvolvimento o padrão é `ambos`: um `npm run dev` atende os dois hosts, sem
gastar uma segunda porta. `bot.localhost` é resolvido pelo Chrome e pelo Firefox
sem tocar em `/etc/hosts`.

**O roteamento é por host, com lista explícita de rotas** — não por prefixo de
caminho:

```ts
if (noHostDoBot) {
  if (ehCaminhoDoRobo(pathname)) return NextResponse.next();
  return NextResponse.redirect(new URL(pathname + search, SITE_URL));
}
if (ehCaminhoDoRobo(pathname)) return NextResponse.redirect(new URL(pathname + search, BOT_URL));
```

Endereço pedido no host errado **leva ao host certo** em vez de dar erro, e cada
tela existe num endereço só — duplicata de URL é o pior resultado possível pra um
site que vive de busca.

### A armadilha do prefixo

A alternativa óbvia é pôr as telas novas em `app/robo/` e reescrever `/painel` →
`/robo/painel`. Não funciona: a reescrita pega **tudo**, inclusive o que o
framework serve por caminho próprio — `/icon.png`, `/opengraph-image.jpg`,
`/robots.txt`, os assets de rota. Cada um vira um 404 silencioso no subdomínio, e
o favicon sumido é o sintoma mais fácil de ignorar.

Lista explícita custa lembrar de somar a rota nova. É barato: esquecer manda o
visitante pro outro host em vez de quebrar a página.

### O `robots.txt` de cada host é outro

O do subdomínio logado proíbe tudo; o do site convida. O gerador do framework não
vê o host — ele produziria um arquivo só para os dois. A saída é responder o do
subdomínio direto no roteador.

## Do lado da plataforma

Dois serviços apontando pro mesmo repositório **redeployam juntos por padrão**, o
que desfaz a separação inteira em silêncio. Restrinja os *watch paths* do serviço
de estado vivo aos diretórios dele. E trave uma réplica só: duas religariam as
mesmas sessões e disputariam a mesma conexão do outro lado.

## Nota de versão

No Next 16 o arquivo é `proxy.ts` e a função exportada se chama `proxy` — a
convenção `middleware.ts` foi depreciada e renomeada. Ele roda em Node, não no
edge, mas ainda executa em toda requisição: o módulo que ele importa tem que ser
barato.

## Conexões
- Princípio: [[Configuração vem do ambiente, não do código]]
- Depende de: [[Processo que guarda conexão viva não tolera deploy frequente, e o log não denuncia]]
- Irmã: [[Railway não roda compose, cada serviço vira uma peça da plataforma]] ·
  [[Cookie de sessão é host-only, www e apex canonizam num domínio só]] ·
  [[App sob subcaminho fica na raiz e o proxy tira o prefixo]]
- Visto em: [[piwdex2]]
- Mapa: [[Infra]]
