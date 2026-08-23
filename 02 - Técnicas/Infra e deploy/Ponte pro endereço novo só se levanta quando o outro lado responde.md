---
tags: [tipo/atomica, camada/padrao, infra, armadilha]
criado: 2026-08-23
---

# Ponte pro endereço novo só se levanta quando o outro lado responde

> Redirect é código, mas o destino dele é um fato de infraestrutura. Escrever
> `/vip → https://bot.exemplo.com/painel` num commit publica a ponte no instante
> do deploy — e se o DNS daquele subdomínio ainda não existe, o que estava
> funcionando (um redirect pra home) vira erro de resolução pra todo mundo.

## O caso

O piwdex mandava `/vip` e `/bot-app` pra home desde que o robô saiu do ar. São
rotas salvas dentro do próprio jogo, no Discord e no favorito de quem usava — o
público que ainda pede o sistema pelo nome antigo. Quando o robô voltou, apontar
os quatro redirects pro subdomínio novo parecia a conclusão natural do trabalho.

Só que o código ficou pronto antes do DNS. Publicar assim trocaria uma home por
um `DNS_PROBE_FINISHED_NXDOMAIN`, justamente para quem tinha o link antigo.

## A saída

A variável de ambiente **é** a declaração de que o outro lado existe:

```ts
const BOT_URL = process.env.NEXT_PUBLIC_BOT_URL?.trim().replace(/\/$/, "") || "";
const DESTINO = BOT_URL ? `${BOT_URL}/painel` : "/";
```

Sem a variável, o redirect antigo continua valendo. Ligar a ponte deixa de ser um
commit e vira um ato de configuração — feito quando o DNS responde, por quem está
olhando pro painel.

Repare que aqui a variável **não tem padrão**, ao contrário do resto do projeto.
É deliberado: em `NEXT_PUBLIC_SITE_URL` o padrão é o endereço canônico, que sempre
existe; aqui o valor ausente significa "ainda não", e um padrão apagaria essa
diferença.

## Detalhe que morde

`redirects()` é avaliado **uma vez, no build**, e gravado no manifesto de rotas.
Preencher a variável e reiniciar não muda nada — precisa rebuildar. Plataformas
que disparam deploy ao alterar variável já fazem o certo; um `restart` manual,
não.

## Vale além de redirect

Qualquer referência a um endereço que ainda não subiu tem a mesma forma: link no
rodapé, `notification_url` de webhook, botão que abre outro serviço. A pergunta é
sempre "o que acontece se isto for publicado antes do outro lado?" — e a resposta
boa é "continua valendo o comportamento anterior".

## Conexões
- Princípio: [[Configuração vem do ambiente, não do código]]
- Irmã: [[Herdar um deploy é herdar o contrato dele, não só o domínio]] ·
  [[Um processo serve dois hosts quando o papel vem do ambiente]] ·
  [[Migração de dados mantém o antigo como reserva até a virada]]
- Visto em: [[piwdex2]]
- Mapa: [[Infra]]
