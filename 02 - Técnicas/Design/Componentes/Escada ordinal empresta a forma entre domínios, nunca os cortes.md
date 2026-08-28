---
tags: [tipo/atomica, camada/padrao, design, armadilha]
criado: 2026-08-28
---

# Escada ordinal empresta a forma entre domínios, nunca os cortes

> Uma escada de juízo — verde, âmbar, vermelho, do bom ao crítico — é feita de duas
> coisas: a **forma** (quantos degraus, em que ordem, com que cores) e os **cortes**
> (onde cada degrau começa). Reusar a escada de outra tela é reusar a forma. Copiar os
> cortes junto é herdar a régua de um mundo que não é o seu.

## O erro

Duas seções mediam "atraso em dias" — a distância entre o fato e o registro. A do
Contábil já tinha a escada pronta e ela era boa: *até 5 dias · 6 a 30 · 31 a 90 · 91 a
180 · mais de 180*. Reusar parecia óbvio, e é o que
[[A mesma grandeza usa a mesma escada nas duas telas]] recomenda.

Só que no Fiscal a mediana do atraso é **25 dias**, não 1: o setor fecha a competência
anterior durante o mês seguinte, e isso é o ciclo normal, não dívida. Com os cortes do
Contábil, o escritório inteiro cairia nos dois primeiros degraus com quase tudo já fora
do verde — e a tela pararia de separar quem está em dia de quem está devendo, que era a
única coisa que ela existia pra fazer.

O primeiro degrau virou **"até 30 dias"**, e a tela diz por quê no subtítulo.

## Por que não é contradição com reusar a escada

As duas notas falam de coisas diferentes, e a distinção é a útil:

- **Escada categórica** (raridade, situação, status): os valores são o mesmo universo
  nas duas telas. Reusar tudo — nomes, ordem, cores — é obrigatório, senão "Épico" muda
  de sentido conforme a rota.
- **Escada ordinal sobre grandeza contínua** (dias, reais, porcentagem): os degraus são
  um corte que ALGUÉM fez sobre uma distribuição. Outra distribuição, outros cortes.

O nome da grandeza engana: "atraso em dias" é a mesma unidade nos dois módulos e não é a
mesma grandeza. Unidade igual não é distribuição igual.

## Na prática

- A forma mora num lugar só (a paleta sequencial, a ordem, o `faixaDe`); cada domínio
  declara **sua** lista de cortes com o mesmo construtor.
- Antes de escolher os cortes, rode a mediana e o p90 do domínio. Se a mediana cai no
  primeiro ou no último degrau, os cortes estão errados — ver
  [[A régua sai da distribuição, não dos extremos]].
- O primeiro degrau tem de ser **o ciclo normal daquele trabalho**, não zero. Verde
  precisa querer dizer "está no ritmo", e não "é impossível".
- O limiar de alerta (o que fica vermelho na tabela) acompanha a escada do domínio, não
  a do vizinho — senão a cor da célula discorda da cor da barra na mesma tela.
- Escreva o porquê do primeiro corte no subtítulo. Sem isso, o próximo a mexer vai achar
  que 30 dias é frouxidão e "consertar" de volta pra 5.

## Conexões
- Princípio: [[A régua sai da distribuição, não dos extremos]]
- Irmã: [[A mesma grandeza usa a mesma escada nas duas telas]] — lá a escada é
  categórica e se reusa inteira; aqui é ordinal e só a forma atravessa.
- Irmã: [[Limiar em grandeza contínua vira degrau, e o degrau decide a ordem]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Design]]
