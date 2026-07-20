---
tags: [tipo/atomica, camada/padrao, dev/backend]
criado: 2026-07-19
---

# Polling substitui webhook quando não há IP público

Webhook é uma chamada **de entrada**: o serviço externo precisa alcançar o teu
servidor. Numa app que roda só na rede interna — IP privado, sem port forward —
esse callback simplesmente nunca chega. E falha em silêncio: nenhum erro, o
registro só fica parado num estado intermediário para sempre.

A inversão resolve: em vez de esperar a notificação, **o servidor pergunta**. Um
cron chama o mesmo endpoint de consulta que o webhook usaria para buscar
detalhes, e concilia o estado. Custo: latência de alguns minutos em vez de
instantânea. Ganho: funciona em qualquer topologia de rede.

Quase toda API que oferece webhook também oferece o `GET` equivalente — no
[[Navecon Controller]] a lib da Autentique já tinha `detalharDocumento()`, usada
pelo próprio webhook para completar o payload. O polling nasceu de graça.

## O desenho que faz os dois conviverem

Não escolha entre webhook e polling — extraia o trabalho para uma função e dê
dois gatilhos a ela:

```
lib/assinatura-sync.ts   -> conciliarPorDocToken(token)
  ├── api/webhooks/...            (instantâneo, precisa de IP público)
  └── api/cron/verificar-...      (atraso de minutos, funciona sempre)
```

O handler do webhook vira ~20 linhas de extrair-ID-e-delegar. Se um dia surgir um
túnel ou IP público, é só agendar o cron mais espaçado — nada mais muda.

**A função precisa ser idempotente**, porque os dois gatilhos podem chegar na
mesma entidade. Guarde a saída pelo estado final, não pelo evento:

```ts
if (todosAssinaram) {
  if (ata.assinaturaStatus === 'ASSINADA') return { mudou: false };  // já processado
  // ... baixa PDF, atualiza, notifica
}
```

Sem essa guarda, o segundo gatilho reenvia o e-mail de "documento assinado".
Idempotência aqui não é elegância — é o que impede spam para o cliente.

## Quando o polling não serve

Se o volume de entidades em aberto for grande, varrer todas periodicamente fica
caro. Filtre pelo estado pendente (`status NOT IN ('ASSINADA','RECUSADA')`), que
costuma ser um conjunto pequeno mesmo em base grande. Se ainda assim pesar, aí a
resposta é expor o webhook — via **Cloudflare Tunnel**, que dá URL HTTPS pública
sem port forward nem IP fixo.

## Conexões
- Princípio: [[Configuração vem do ambiente, não do código]]
- Visto em: [[Navecon Controller]]
- Mesma família de problema: [[Plataforma de IA hospedada prende o app pelo banco]]
- Mapa: [[Backend]]
