---
tags: [tipo/atomica, camada/principio, design]
criado: 2026-08-29
---

# Catálogo de componentes é contrato vivo, não documentação

> A rota onde cada peça aparece rodando, no build de produção, antes de existir
> tela que a use. Escrita no fim, ela é inventário morto; escrita junto, é o
> lugar onde as decisões de interface ficam guardadas com o motivo.

## O que o catálogo evita

Um sistema com quatro superfícies (vitrine, painel, moderação, produto pago) é
escrito em meses diferentes, e quem escreve a quarta não lembra do que existia
na primeira. Sem catálogo, o segundo botão de "confirmar" nasce ligeiramente
diferente do primeiro, e a partir daí a diferença se propaga: cada tela nova
copia a versão mais próxima em vez da versão certa.

O componente que existe no código mas não aparece no catálogo será reescrito
por quem não sabia que ele existia. É por isso que a regra é entrar **no mesmo
commit** em que a peça nasce, e não numa passada de documentação depois.

## O bloco diz o porquê, não só mostra a peça

Catálogo que só exibe é galeria. O que evita retrabalho é a frase que explica a
decisão embutida: "a célula reserva a linha do rótulo mesmo sem rótulo, senão a
fila desalinha e alinhar pela base só disfarça". Sem ela, a próxima pessoa
remove a linha vazia por achar que é sobra, e o desalinhamento volta.

## Estado que só existe aberto se mostra aberto

Modal, gaveta, menu suspenso, dica e torrada só aparecem depois de um gesto.
Se o catálogo mostra apenas o botão que dispara, quem olha conclui que a peça
não existe, e a conclusão é razoável: ele não tem como saber a diferença entre
"não construído" e "construído mas escondido atrás deste clique".

Cada um desses precisa de duas formas na página: a viva, atrás do gatilho, e a
estática, sempre visível. A estática é o que faz o catálogo ser conferível de
relance e por captura de tela.

## A fronteira de domínio mora na pasta

Separar o que conhece o domínio do que não conhece é o que permite reusar a
base nas outras superfícies. Um botão não sabe o que é anúncio; um cartão de
anúncio sabe. Quando a fronteira não é explícita, o primitivo acumula regra de
uma tela e deixa de servir às outras sem uma condição nova a cada uso.

## Onde a régua fica frouxa

Projeto sem interface não precisa disso, e um script de uma tela também não. A
conta vira positiva quando existe mais de uma superfície ou mais de um mês de
trabalho: aí o custo de manter o catálogo é menor que o custo de descobrir, na
quinta tela, que existem três botões primários diferentes.

## Conexões
- Irmã: [[O primitivo só padroniza o que passa por dentro dele]] ·
  [[A variante de um controle muda a intenção, não o tamanho]] ·
  [[Todo estado da tela tem visual]]
- Visto em: [[Privello]]
- Mapa: [[Base]] · [[Design]]
