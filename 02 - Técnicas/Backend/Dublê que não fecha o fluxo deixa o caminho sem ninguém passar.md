---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-09-17
---

# Dublê que não fecha o fluxo deixa o caminho sem ninguém passar

> O dublê de um fornecedor externo existe para o sistema rodar antes de a conta
> real existir. Se ele consegue COMEÇAR o fluxo mas nunca o termina, o trecho
> depois dele nunca é percorrido — e esse trecho costuma ser justamente onde o
> produto entrega o que vendeu.

## O problema

O dublê de pagamento do [[telebot]] nasceu assim:

```ts
async criarPix(pedido) { /* devolve um código de teste */ }
async consultar()      { return "aguardando"; }   // sempre
```

Parecia conservador e honesto: ele não inventa que alguém pagou. Um comentário
no arquivo dizia que a confirmação viria de um botão na tela do pedido — botão
que não existia.

O efeito é pior do que uma peça faltando, e é diferente de
[[Peça de mentira que não se anuncia vira fundação de coisa real]]. Lá o falso
**afirma** algo que não aconteceu. Aqui ninguém mente: o fluxo simplesmente
para, e por meses a metade seguinte — confirmar, criar o acesso, creditar a
carteira, gerar o convite, avisar o comprador — não foi percorrida por ninguém
nenhuma vez. O código estava escrito, tinha passado no build e no tipo, e nunca
tinha rodado.

Quando a credencial real chegar, a primeira execução daquele caminho vai
acontecer com dinheiro de verdade em cima.

## A solução

**No modo simulado, perguntar é pagar.**

```ts
async consultar() { return "pago"; }
```

Quem chamou já decidiu perguntar; sem dinheiro real envolvido, não há o que
perder. O risco de errar para o lado permissivo aqui é nenhum, e o risco do lado
conservador é o caminho inteiro ficar sem teste.

O que importa é que as duas confirmações — a do dublê e a do provedor real —
passem pelo **mesmo** código: a mesma rota de notificação, a mesma transação de
confirmação, a mesma entrega. Um atalho paralelo "só para desenvolvimento"
devolveria o problema com outra roupa: o caminho exercitado deixaria de ser o
caminho que roda em produção.

## O teste que decide

Pergunte do dublê: **o que ainda não rodou nenhuma vez por causa dele?** Se a
resposta não for "nada", ele está curto.

Vale além de pagamento: dublê de e-mail que enfileira e nunca "entrega" esconde
o que acontece depois do envio; dublê de upload que aceita o arquivo e não
devolve URL esconde a tela que mostra o anexo.

## O que continua valendo da nota irmã

O dublê tem que **se anunciar**. No telebot ele carrega `simulado: true`, e a
interface carimba um aviso em toda tela onde a cobrança aparece; o código
copiável não é um Pix válido, então quem tentar pagar recebe erro do banco em
vez de perder dinheiro.

## Conexões
- Princípio: [[Fornecedor externo entra pelo contrato do app, não o app pelo dele]]
- Irmã: [[Peça de mentira que não se anuncia vira fundação de coisa real]] · [[Verificar no build de produção, não só em dev]]
- Visto em: [[telebot]]
- Mapa: [[Backend]] · [[Base]]
