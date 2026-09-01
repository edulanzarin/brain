---
tags: [tipo/atomica, camada/padrao, design]
criado: 2026-08-30
---

# Fila de decisão agrupa pelo alvo, não pelo aviso

> Numa fila onde a pessoa DECIDE sobre algo — moderar um anúncio, atender um
> chamado, tratar um alerta —, a linha é o alvo da decisão, não cada aviso que
> chegou sobre ele. Sete denúncias do mesmo perfil são uma decisão, não sete.

## O que uma linha por aviso estraga

**Repete o julgamento.** A mesma pessoa aparece sete vezes seguidas, e quem
modera abre o mesmo perfil sete vezes para responder a mesma pergunta.

**Some com o sinal mais forte que a tela tem.** O volume — sete avisos contra um
— é a informação que mais deveria pesar na decisão, e espalhado em sete linhas
ele vira só comprimento de lista.

**Deixa metade resolvida.** Resolver seis e esquecer a sétima é um estado que só
existe porque a fila permitiu, e ele volta amanhã como caso novo.

## A ordem também muda

Ordenada pela mais nova, o caso grave de ontem afunda sob os avisos de hoje.
Numa fila de decisão a ordem sai da **gravidade**, e quantidade é a
aproximação mais barata dela.

É o inverso da fila de conferência, que se ordena pela mais antiga — lá cada
item é uma pessoa esperando resposta, e o critério é não deixar ninguém no
fundo. As duas filas moram no mesmo painel e ordenam ao contrário uma da outra,
de propósito.

## Vale lembrar

Agrupado, cada ação vale para o grupo inteiro: resolver fecha todos os avisos
daquele alvo de uma vez. E a ação que muda o alvo (suspender) precisa fechar os
avisos junto — senão o mesmo item volta ao topo amanhã, e a próxima pessoa
refaz o trabalho já feito.

O agrupamento costuma ficar melhor no código que lê do que numa consulta
agregada: `groupBy` devolve a contagem e exige uma segunda ida ao banco pelos
textos, que é justamente o que quem decide lê.

## Conexões
- Princípio: [[Ordene pela grandeza que decide, não pela que impressiona]]
- Irmã: [[Uma pendência de prazo fecha por ato explícito, não por sinal inferido]]
- Visto em: [[Privello]]
- Mapa: [[Design]]
