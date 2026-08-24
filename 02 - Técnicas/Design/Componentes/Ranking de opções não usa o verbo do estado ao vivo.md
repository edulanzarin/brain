---
tags: [tipo/atomica, camada/padrao, design, armadilha]
criado: 2026-08-24
---

# Ranking de opções não usa o verbo do estado ao vivo

> Uma lista ordenada de possibilidades e um painel de status se parecem: as
> mesmas linhas, os mesmos ícones, os mesmos números. O que os separa é o VERBO —
> e quando a lista de opções escreve no presente do indicativo, ela deixa de
> oferecer e passa a afirmar.

## O caso

No [[piwdex2]] o piloto do robô mostra, para cada pokémon do time, onde ele
renderia mais por hora — seis linhas, ordenadas do melhor pro pior. Cada linha
dizia:

```
Golem      nv 409   caçando Brave Charizard   seguro   294k/h
Electrode  nv 232   caçando Mantine           seguro   161k/h
Primeape   nv 116   caçando Pikachu           seguro    43k/h
```

O Eduardo leu e perguntou: **"como que cada um tá caçando um? dá pra fazer
isso?"**. Não dá — o jogo entrega uma sessão, com um líder, em um campo. A lista
era um ranking de PARES ("com qual dos meus, e onde"), e só a primeira estava
acontecendo.

Nada ali estava tecnicamente errado. O ranking era o certo, a ordem era a certa,
os números eram os certos. Só o verbo mentia, seis vezes.

## O que separa oferta de fato

- **O verbo.** "em Mantine" descreve possibilidade; "caçando Mantine" descreve
  acontecimento. Trocar uma palavra por linha resolve mais que qualquer legenda.
- **A posição visível.** Numerar (1, 2, 3) declara que aquilo é uma ORDEM. Sem o
  número, uma lista ordenada é indistinguível de uma lista de coisas coexistindo.
- **O que está no ar ganha rótulo, não só realce.** Uma borda mais clara só
  funciona por comparação — quem olha uma linha por vez não a percebe. Um "no ar"
  escrito é lido isolado.
- **Uma frase de enquadramento antes da lista**, não depois: "um de cada vez" no
  topo evita a leitura errada; embaixo, corrige uma leitura que já aconteceu.

## O rótulo sai do estado, nunca da posição

Marcar "o primeiro é o que está rodando" é uma coincidência que vira bug: em
sistema que reavalia, o ranking muda antes da troca acontecer. O item no topo
passa a ser o candidato NOVO enquanto o motor ainda executa o antigo — e o
destaque aponta pra linha errada exatamente no instante mais confuso, que é
durante a mudança.

Case o rótulo com o identificador do que está executando (`slug === estado.slug`),
não com o índice. Se nenhum casar, nenhuma linha se destaca — e isso também é uma
informação verdadeira.

## A mesma pedra sem lista nenhuma

Não é só sobre listas: qualquer rótulo que traduz um ESTADO pode acabar afirmando
um fato que o estado não carrega. No mesmo sistema, ligar o robô mostrava
"CAÇANDO há 22s" sem caçada escolhida — e a tela se desmentia três linhas abaixo,
com "escolha uma caçada acima".

O rótulo sobrou de uma separação anterior: `rodando` deixou de significar
"caçando" e passou a significar "a sessão de jogo é sua", e o mapa de traduções
não acompanhou. **Quando um estado se divide em dois, o vocabulário da tela é a
última coisa a ser revisada — e a primeira a mentir.** O sinal é a própria tela
se contradizendo: se duas partes dizem coisas opostas, uma delas está traduzindo
um estado que não existe mais.

## Conexões
- Princípio: [[Ver o plano e mandar executar são duas ações]]
- Irmã: [[Todo estado da tela tem visual]] ·
  [[Estimativa fraca informa, número verificado ordena]] ·
  [[Tier é nota com régua fixa, não posição na fila]]
- Visto em: [[piwdex2]] — duas vezes no mesmo dia: a lista do piloto, e o crachá do topo
- Mapa: [[Design]]
