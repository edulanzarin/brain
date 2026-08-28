---
tags: [tipo/atomica, camada/principio, armadilha]
criado: 2026-08-27
---

# Razão só afirma quando os dois lados vêm do mesmo trabalho

> Dividir dois números que a tela tem à mão é fácil e quase sempre errado. A razão só mede alguma coisa se numerador e denominador nascerem da mesma população e do mesmo esforço; senão ela é aritmética válida afirmando um fato falso.

## A regra

Antes de publicar uma divisão, faça duas perguntas:

1. **Os dois lados falam da mesma população?** Numerador de uma pessoa sobre
   denominador do time inteiro não é participação nem taxa — é um número sem nome.
2. **Os dois lados custam o mesmo tipo de trabalho?** Volume produzido por máquina
   sobre tempo gasto por gente mede a máquina e cobra da gente.

Se qualquer das duas falhar, ou se troca um dos lados, ou se troca a pergunta.

## Por que

Aba de tempo de um placar de produtividade: 1.959.571 lançamentos contábeis divididos
por 2.840 horas de uso do sistema deram **690 lançamentos por hora**. A conta está
certa e a afirmação é falsa: quase todo o numerador é importação fiscal em lote, que
grava dezenas de milhares de linhas sem consumir um minuto humano. Publicado como KPI,
o número ou parece bug ou vira meta — as duas leituras ruins. Trocado por **horas por
empresa**, que é a pergunta que a fonte responde bem, o mesmo dado passou a dizer
quanto de atenção cada cliente custa.

O segundo caso, na tela vizinha: "regravação = exclusões ÷ lançamentos do período" é
uma boa taxa para o escritório. Quando a tela isola UMA pessoa, o numerador vira o dela
e o denominador segue do time — a mesma fórmula, agora sem significado. O conserto não
foi arrumar a fórmula: foi **trocar de indicador** quando o recorte muda, para
participação no total de exclusões.

## Na prática

- Nomeie a razão em voz alta antes de codificar: "lançamentos por hora **humana**"
  denuncia sozinho que o numerador não é humano.
- Recorte mudou, indicador muda. Uma fórmula que atravessa "time" e "pessoa" sem mexer
  em nada está mentindo num dos dois estados.
- Razão que sobrevive por comparação relativa (ranking entre pessoas) pode ficar, mas
  como coluna com aviso, nunca como número grande no topo.
- Mostrar os operandos ao lado do resultado é o antídoto barato —
  [[Medidor de razão nomeia a grandeza e mostra os operandos]].

## Conexões
- Irmã: [[Ausência só aparece contra o universo, nunca contra a tabela de eventos]]
- Irmã: [[Rendimento é vazão vezes tempo em pé, não vazão de pico]]
- Técnica que aplica: [[Medidor de razão nomeia a grandeza e mostra os operandos]]
- Mapa: [[Base]]
