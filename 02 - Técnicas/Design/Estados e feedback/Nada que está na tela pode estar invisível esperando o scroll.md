---
tags: [tipo/atomica, camada/padrao, design, dev/frontend, armadilha]
criado: 2026-08-24
---

# Nada que está na tela pode estar invisível esperando o scroll

> Revelação no scroll tem uma regra e ela é binária: quando um bloco entra na
> área de leitura, ele **já terminou** de aparecer. Se o gatilho dispara *depois*
> de o bloco entrar, existe uma faixa da tela — sempre a de baixo, sempre a que a
> pessoa está lendo — ocupada por conteúdo transparente.

## O gatilho que parece cedo e é tarde

Os dois números do `IntersectionObserver` decidem isso, e é fácil escrever o
oposto do que se quis:

```js
// ERRADO — a raiz ENCOLHE: dispara mais TARDE
{ threshold: 0.15, rootMargin: "0px 0px -12% 0px" }

// CERTO — a raiz se estende pra FORA da janela: termina antes de entrar
{ threshold: 0,    rootMargin: "0px 0px 18% 0px" }
```

`rootMargin` negativo embaixo puxa o fim da raiz pra cima. O comentário no código
real dizia "dispara ANTES de entrar de fato" enquanto o valor fazia o contrário —
e ninguém confere um comentário contra um sinal.

**Limiar por fração não serve pra bloco alto**, e essa é a metade que morde. Ele
exige *mais pixels* quanto maior a peça: com `threshold: 0.15`, um bloco de 310px
precisa de 46px dentro da raiz; um de 800px precisa de 120px. A peça que mais
precisa aparecer cedo é justamente a que aparece mais tarde. Com `threshold: 0` o
gatilho passa a depender só de onde o bloco está, que é a pergunta real.

## A janela baixa é onde isso aparece

Em monitor alto o defeito é invisível: a faixa inteira entra em cena de uma vez e
não há posição de rolagem que mostre o buraco. Em notebook — ~470px de altura
útil — o mesmo código deixava mais de cem pixels do rodapé da tela permanentemente
em `opacity: 0`.

Regra prática: teste revelação na **janela mais baixa** que o layout suporta, não
na do seu monitor. Varrer a página de 120 em 120px e afirmar *"nenhum bloco dentro
dos 75% de cima da janela está abaixo de 0,5 de opacidade"* vira um teste que roda
em segundos e reprova exatamente esse caso.

## Revelação é por unidade de SIGNIFICADO

O segundo defeito não é de número, é de recorte. Uma faixa de apresentação tinha
duas revelações — uma no texto, outra na arte — cada uma com seu observador. Elas
são uma composição: o título nomeia a peça que está do lado. Com gatilhos
independentes existia posição de rolagem em que a arte enchia a tela com o texto
da mesma faixa ainda apagado, e a página lia como arte solta sobre o fundo.

O que se revela junto é o que só significa junto. Se duas metades podem aparecer
separadas sem prejuízo, são duas peças; se uma sem a outra vira enigma, é uma.

## O que mais vale lembrar

**Nascer visível e ligar o efeito depois.** O bloco renderiza opaco e só recebe
`opacity: 0` quando o cliente confirma que vai animar. O contrário — nascer
invisível — deixa a página em branco sem JavaScript e entrega página vazia pro
rastreador. É o mesmo argumento de
[[Conteúdo do servidor não pode nascer invisível esperando o cliente]], aplicado a
uma animação em vez de a um carregamento.

**O teste de "já está em cena" olha as duas bordas.** Só `top < innerHeight`
deixa passar quem chegou por link com âncora ou recarregou no meio da página: o
bloco tem `top` negativo com o rodapé à mostra, e fica esperando um observador pra
revelar coisa que já está na cara. A condição é `top < innerHeight && bottom > 0`.

## Conexões
- Princípio: [[Todo estado da tela tem visual]]
- Irmã: [[Conteúdo do servidor não pode nascer invisível esperando o cliente]] ·
  [[Reduzir movimento tem que zerar o atraso, não só a duração]] ·
  [[Animação de enfeite escolhe a propriedade pelo custo, não pelo efeito]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]]
