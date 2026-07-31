---
tags: [tipo/atomica, camada/padrao, dev/backend]
criado: 2026-07-31
---

# Fluxo de fechamento é orquestração dos motores que já existem

> Uma tela que responde "posso fechar / posso liberar / está pronto?" não é um
> motor novo — é a **orquestração** dos motores já validados, traduzida num
> veredito. Construir um cálculo próprio pro fechamento duplica lógica e cria uma
> segunda fonte da verdade que vai divergir da primeira.

## O atrito

Depois que um sistema acumula ferramentas isoladas (uma confere isso, outra
calcula aquilo), aparece a vontade de um "resumão" que diga se está tudo certo. A
tentação é escrever de novo, ali, uma versão enxuta de cada checagem — "só o
número que eu preciso". Aí existem **duas** implementações da mesma regra: a da
tela detalhada e a do resumo. Elas divergem no primeiro caso de borda, e o resumo
começa a mentir (verde enquanto a tela detalhada acusa problema).

## O padrão

O fluxo de fechamento **chama os motores que já existem** e só decide a cor:

- Cada checagem reusa a função que a tela dona já usa (o mesmo coletor, a mesma
  conferência). Se o núcleo está preso numa rota, **extrai pra um lib na hora** —
  a rota e o fechamento passam a compartilhar (ver [[O que dois módulos
  compartilham é a query, não a rota]]).
- O orquestrador só mapeia resultado → status (verde/amarelo/vermelho) e agrega o
  veredito geral (o pior). Nenhuma regra de negócio nova mora nele.
- Cada item do veredito **linka a tela onde a pendência se resolve**, já filtrada
  — o resumo é porta de entrada, não beco.

Assim a fonte da verdade continua única: mudou a regra num motor, o fechamento
acompanha de graça. E é barato — nenhum cálculo novo, só releitura do que já roda.

## Por que importa

O valor de um fechamento não está em recalcular; está em **amarrar num veredito**
o que estava espalhado. Recalcular troca essa cola barata por uma dívida: duas
verdades que precisam ser mantidas iguais para sempre.

## Conexões
- Reusa a extração: [[O que dois módulos compartilham é a query, não a rota]]
- Parente na honestidade do número: [[Deixar o método da conferência visível quando o SQL não foi validado]]
- Visto em: [[Navetech Hub]] — Contábil, seção Fechamento: semáforo "posso fechar o mês?" orquestrando balancete + conferência, sem motor novo.
- Mapa: [[Backend]]
