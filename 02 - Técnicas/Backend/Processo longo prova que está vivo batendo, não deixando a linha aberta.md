---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-28
---

# Processo longo prova que está vivo batendo, não deixando a linha aberta

> Um trabalho demorado costuma registrar uma linha ao começar e fechá-la ao
> terminar. Aí "está rodando?" vira "existe linha sem fim?" — e essa pergunta
> responde errado exatamente quando importa: o processo que MORREU também deixa a
> linha aberta. Ausência de fim não é sinal de vida.

## O estrago

Enquanto o sistema acreditar na linha aberta, ele afirma que há trabalho em
andamento onde não há. As consequências se somam:

- a tela mostra progresso congelado como se fosse lento, não morto;
- o botão de iniciar fica indisponível, porque "já tem um rodando";
- um pedido de parada nunca é lido por ninguém, e a interface fica em
  "Parando…" para sempre.

E a causa mais comum de morte não é falha: é **deploy**. Todo `docker compose up
-d --build` mata o que estava no meio. No caso medido, uma janela de abandono de
8 horas significava que subir uma versão travava o módulo pelo resto do dia.

## A regra

O processo **carimba um instante a cada passo**, e quem lê considera vivo só quem
bateu há pouco. A janela é o maior silêncio legítimo com folga — não uma
estimativa da duração total, que é o erro que produz a janela de horas.

Dois cuidados que só aparecem implementando:

1. **A fase silenciosa também bate.** Costuma haver um preâmbulo que não produz
   nada observável (montar a lista antes de percorrê-la). Sem carimbo ali, o
   trabalho parece morto justamente ao começar, e a janela precisaria ser
   esticada até cobrir a preparação — devolvendo o problema.
2. **Parar não é só pedir.** Quem não bate não vai ler pedido nenhum: a ação de
   parar fecha a execução morta na hora e só entrega a bandeira para a viva.

O carimbo cabe no mesmo UPDATE que já grava o progresso, então custa zero ida a
mais ao banco. E preservar o ponto de parada junto é o que transforma um deploy
no meio do trabalho em atraso, não em recomeço.

## Conexões
- Princípio: [[Contador que conta sucesso de promessa afirma que deu certo]]
- Irmã: [[Comando sem resposta precisa de vigia, não de fé]] · [[Confirme a mutação pelo estado que ela deixa, não pelo ack que pode não chegar]] · [[Agenda recorrente é um serviço do compose, não um crontab do host]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Backend]]
