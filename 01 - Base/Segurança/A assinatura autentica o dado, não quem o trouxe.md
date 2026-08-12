---
tags: [tipo/atomica, camada/principio, seguranca]
criado: 2026-08-12
---

# A assinatura autentica o dado, não quem o trouxe

> Um dado que chega pela borda não é confiável pela origem aparente — o IP, o
> cabeçalho, "veio do meu cookie". É confiável quando carrega uma **prova que só o
> detentor do segredo poderia produzir**: uma assinatura (HMAC). Verifique a
> assinatura e você pode aceitar o dado de qualquer portador, inclusive do próprio
> cliente, sem guardar estado e sem confiar no canal.

## A regra

Não pergunte "de quem veio isto?". Pergunte "isto está assinado com o segredo que
só eu tenho?". A resposta separa dado forjável de dado autêntico, independente de
quem o entregou. O corolário poderoso: você pode **devolver estado ao cliente e
reaceitá-lo depois** — se a assinatura bate, o cliente não adulterou.

## Por que

O reflexo é confiar no portador: aceitar o corpo do webhook porque chegou na URL
secreta, tratar o cookie como verdadeiro porque o navegador o mandou, liberar
porque o front diz que o usuário é admin. Todos são forjáveis — a URL vaza, o
cookie se edita, o front mente. A assinatura move a confiança do **canal** (frágil)
para a **posse do segredo** (o que você realmente controla).

## Na prática

- HMAC-SHA256 com um segredo do ambiente; comparar em **tempo constante**
  (`timingSafeEqual`), nunca com `===`.
- Assine tudo que a decisão depende: no cookie de sessão, a expiração entra no que
  é assinado (`exp.hmac(exp)`), senão o cliente estende a própria sessão.
- Verifique **antes** de usar o dado, não depois.
- A assinatura prova autenticidade e integridade, não confidencialidade — não
  guarde segredo no que é só assinado (um cookie assinado ainda é legível).

## Onde já apareceu (dois casos, mesma lição)

- **Webhook de terceiro**: cada payload é aceito porque a assinatura do provedor
  bate, não porque chegou na URL. Ver
  [[Webhook de terceiro se valida pela assinatura antes de confiar no corpo]].
- **Sessão sem estado**: o cookie de sessão do painel é `exp.hmac` — o servidor
  confia nele porque a assinatura fecha, sem tabela de sessões. Ver
  [[Sessão de painel interno é um cookie assinado, não uma tabela de sessões]].

## Conexões
- Irmã: [[Permissão se valida no servidor, não na interface]]
- Técnica que aplica: [[Webhook de terceiro se valida pela assinatura antes de confiar no corpo]] · [[Sessão de painel interno é um cookie assinado, não uma tabela de sessões]]
- Mapa: [[Base]]
