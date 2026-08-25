---
tags: [tipo/atomica, camada/principio, armadilha]
criado: 2026-08-25
---

# Desgaste constante contra recuperação constante é bimodal

> Um recurso que perde a uma taxa fixa e ganha a outra taxa fixa não tem
> meio-termo: ou a recuperação vence e o recurso fica cheio pra sempre, ou o
> desgaste vence e ele zera, mais cedo ou mais tarde. Mexer nas duas taxas só
> **move a fronteira**, nunca cria a faixa do meio. A faixa do meio precisa de um
> terceiro elemento que responda ao estado.

## O problema

A intuição diz que existe um ajuste fino entre gastar e repor: taxa um pouco
maior, sobrevive; um pouco menor, não. Existe — mas é um ponto, não uma faixa.
De um lado da fronteira o sistema é imortal, do outro ele morre. Não há
"aguenta mais ou menos".

Isso aparece em qualquer par drenagem/reposição: vida contra cura, crédito contra
recarga, fila contra vazão, orçamento contra receita.

O sintoma que denuncia é diagnóstico: você varre a taxa de recuperação em três
valores e os três dão o mesmo veredito qualitativo, só mudando o tempo até
acontecer. Se dobrar o parâmetro não muda a natureza do resultado, o parâmetro
não é a alavanca.

## A saída

Um **terceiro elemento que responde ao estado**, e não ao tempo. O mais simples é
um estoque consumível gasto quando o recurso cai abaixo de um limiar:

- enquanto há estoque, o sistema se segura;
- quando o estoque acaba, ele cede.

Isso troca a pergunta. Não é mais "quanto tempo aguenta", que era binário: é
"quanto de estoque preciso levar", que é uma decisão do usuário, mensurável e
com preço. O limite deixa de ser uma constante escondida na fórmula e vira
recurso visível.

Vale a mesma leitura para retry com orçamento, cota que pode ser comprada e
buffer que se enche antes do pico.

## O que mais vale lembrar

Se você já mexeu na taxa três vezes e o comportamento continua igual em espécie,
pare de mexer: o problema não é o valor, é a forma do sistema.

## Conexões
- Irmã: [[Limiar em grandeza contínua vira degrau, e o degrau decide a ordem]]
- Visto em: [[Vespéria]]
- Mapa: [[Base]]
