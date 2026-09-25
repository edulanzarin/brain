---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-09-25
---

# Hora formatada no servidor sai no fuso do container e quebra a hidratação

> Componente "client" do Next também renderiza no servidor. Um formatador de
> data e hora sem `timeZone` escreve no fuso de quem executa: o container, em
> UTC, manda "13:02" no HTML, o navegador em São Paulo desenha "10:02", e o
> React acusa erro de hidratação (418). Na máquina de dev, no mesmo fuso do
> navegador, os dois textos coincidem e o erro não existe.

## Os dois textos que chegam

Pôr `timeZone: "America/Sao_Paulo"` no `Intl.DateTimeFormat` conserta metade,
e a outra metade quebra no lugar. O app recebe hora em duas formas:

| forma | de onde vem | o que é |
|---|---|---|
| com fuso: `2026-09-10T13:02:00.000Z` | `Date` do driver `pg` serializado em JSON | um instante |
| sem fuso: `2026-09-24T15:42:00` | `to_char` sem offset, hora do Questor | a hora do relógio de quem registrou |

`new Date("2026-09-24T15:42:00")` lê o texto sem fuso como hora LOCAL de quem
executa: no navegador vira 15:42 de São Paulo, no container vira 15:42 UTC, que
formatado em São Paulo sai 12:42. Antes da correção esse tipo coincidia por
acaso (UTC lido e escrito em UTC); com o fuso explícito, passa a divergir.

## A correção

- Com fuso: formatar o instante com `timeZone` explícito.
- Sem fuso: não passar pelo `Date`. O texto já é a hora certa; recortar dia,
  mês, ano, hora e minuto e montar o texto na mão.

Um teste fixa a saída com o processo em UTC e em São Paulo
(`process.env.TZ` muda o fuso do Node em tempo de execução).

## Como achar

A mensagem minificada não diz o nó. Buscar o HTML do servidor com `fetch` (a
mesma página, com o cookie de sessão), abrir no headless e comparar o texto de
cada bloco contra o DOM depois da hidratação, ignorando `&nbsp;` contra espaço
e gráfico que só desenha no cliente. Divergência de exatamente 3 horas é fuso.

## Conexões
- Irmã: [[Postgres de container nasce em UTC, e a hora formatada sem fuso mente]]
  (o banco ao lado, mesmo fuso herdado; é de lá que vem o texto sem offset)
- Irmã: [[Agendador em container conta as horas em UTC]] (o processo Node do agendador)
- Irmã: [[Notação compacta do Intl muda com o ICU e quebra a hidratação]]
  (o outro 418 que só aparece no build, e o mesmo jeito de achar)
- Princípio: [[Verificar no build de produção, não só em dev]]
- Visto em: [[NaveX]]
- Mapa: [[Frontend]]
