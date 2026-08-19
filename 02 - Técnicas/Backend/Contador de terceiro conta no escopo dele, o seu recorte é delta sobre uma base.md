---
tags: [tipo/atomica, camada/padrao, dev/backend, dados, armadilha]
criado: 2026-08-19
---

# Contador de terceiro conta no escopo dele, o seu recorte é delta sobre uma base

> O contador que vem de fora (analyzer da sessão, quota da API, odômetro do device) é
> cumulativo **no escopo de quem o emite** — quase sempre a conexão ou o login, não a
> unidade de trabalho que o seu produto mostra. Exibir o valor cru como se fosse o seu
> recorte cola os números do trecho anterior no trecho novo.

## O problema

Você troca de unidade (a caçada, a campanha, o turno) sem derrubar a conexão, zera o seu
estado local e o próximo frame da fonte volta com o acumulado inteiro por cima. A tela
mostra a unidade nova com o tempo, os kills e as taxas da unidade velha — e quem olha jura
que a troca não aconteceu.

Pior que o visual: se ao fechar cada unidade você **persiste** esse valor num totalizador,
cada troca relança o acumulado inteiro. Três trocas numa conexão viram três vezes o mesmo
número no histórico. É dupla-contagem silenciosa, e o dashboard não tem como perceber.

A raiz é uma suposição não testada: "entrar de novo no campo/fluxo zera o contador". Zerar
é evento da fonte, não seu. Reler a documentação do protocolo costuma dizer o contrário —
no Poke Idle World está escrito que o analyzer é da **sessão** e só zera desconectando.

## A solução

No marco zero da SUA unidade, guarde uma **base** (o valor do contador naquele instante) e
exponha sempre `valor − base`. A base é privada: a UI, o resumo e os totais nunca veem o
cru.

Três detalhes que decidem se funciona:

- **Só o que acumula se subtrai.** Saldo e taxas são derivados e precisam ser
  **recalculados** em cima do delta (`saldo = ganho − custo`, `taxa = total / horas do
  trecho`), nunca subtraídos. Lista cumulativa (drops por item) se diferencia item a item,
  descartando o que ficou em zero.
- **A base é o primeiro valor DEPOIS do começo, não o último de antes.** Entre parar e
  recomeçar, o contador da fonte pode ter andado sozinho; adotar o próximo frame como
  marco zero perde milissegundos e resolve isso de graça.
- **Valor menor que a base = a fonte zerou** (reconectou, virou o dia, o device reiniciou).
  Derrube a base nesse instante. Com essa guarda o código fica certo nos dois mundos — se a
  fonte zerar ou não no seu evento, você não precisa saber.

```ts
if (rebase) { base = raw; rebase = false; }           // marco zero da unidade nova
else if (base && SUMS.some(k => raw[k] < base[k])) base = null; // a fonte zerou sozinha
view = delta(raw, base);
```

## O que mais vale lembrar

Antes de "zerar o estado local", pergunte de quem é o contador. Estado que você calcula
zera quando você manda; estado que chega de fora zera quando a fonte quer — e o único jeito
honesto de recortar o que é seu é medir a diferença entre duas marcas.

## Visto em

No piwdex o robô segura a sessão do jogo e troca de hunt sem reconectar. Sair de uma hunt
manual e ligar o AUTO mantinha 3.703 derrotados e 6h55m na hunt nova, e cada troca de faixa
do cérebro relançava o acumulado em `robot_sales`. A sessão passou a guardar
`analyzerBase` e a expor só o delta.

## Conexões
- Princípio: [[Balancete é movimento do período, saldo é consequência]]
- Irmã: [[Total ao vivo é o persistido fechado mais o em andamento ainda não gravado]]
- Parente: [[Config que a sessão cacheia no init não vê a escrita no backend, reaplique na mesma conexão]]
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
