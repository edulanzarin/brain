---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-25
---

# Multiplicador de contexto entra depois da razão, não dentro dela

> Quando uma fórmula compara duas forças por razão — `a / (a + b)`, `a² / (a² + b²)`
> — o que entra na razão é a grandeza **crua** dos dois lados. Bônus de contexto
> (vantagem, promoção, peso da situação) multiplica o **resultado**, nunca um dos
> lados de dentro. Colocar o bônus dentro infla um dos pratos da balança e o
> resultado deixa de medir força relativa.

## O problema

A razão só é honesta se os dois lados estiverem na mesma régua. Se `a` já vem
multiplicado por um fator de contexto e `b` vem cru, `a / (a + b)` não responde
mais "quem é mais forte": responde "quem é mais forte depois de eu ter mexido só
num lado".

O estrago é maior do que parece porque a razão **satura**. Inflar `a` em 3x não
aumenta o resultado em 3x — empurra a razão pra perto de 1 e apaga a diferença
que ela existia pra medir. Um lado fraco com bônus alto passa a parecer
equivalente a um lado forte sem bônus.

## O caso concreto

Num motor de combate, a mitigação de dano era `off² / (off² + def²)`, e `off` já
vinha com o multiplicador de tipo embutido (até 3x, somando vantagem elemental e
bônus do mesmo tipo). O resultado: um ataque cru de 10 entrava na razão valendo
30 e enfrentava uma defesa crua de 14. A mitigação, que deveria proteger quem
tinha o dobro de defesa, entregava 82% do dano.

O sintoma na tela: um inimigo de nível 5 derrubava um time de nível 20. E o teste
que existia não pegava, porque testava a razão com números já inflados dos dois
lados.

```
// errado: o tipo desloca a mitigação
dps = (off * tipo) / DIV * ((off * tipo)² / ((off * tipo)² + def²))

// certo: atributo decide a mitigação, o tipo multiplica o dano
dps = off / DIV * (off² / (off² + def²)) * tipo
```

## O que mais vale lembrar

O teste que denuncia é comparar **duas escalas distantes**: com força parecida as
duas formas dão quase o mesmo número, e é por isso que o erro sobrevive à
conferência. Ver [[Calibre nas pontas, o meio esconde o defeito]].

A razão quadrática vale a troca quando se quer que a diferença pese: em força
igual ela dá os mesmos 50% da linear, mas derruba muito mais rápido quem está
abaixo. É o que faz "subir de nível" significar parar de apanhar no lugar antigo.

## Conexões
- Princípio: folha isolada — nenhum princípio da Base cobre ainda. Candidato:
  "grandeza só se compara com outra na mesma régua", que também explicaria
  [[Contador de terceiro conta no escopo dele, o seu recorte é delta sobre uma base]].
- Irmã: [[Calibre nas pontas, o meio esconde o defeito]]
- Visto em: [[Vespéria]]
- Mapa: [[Backend]]
