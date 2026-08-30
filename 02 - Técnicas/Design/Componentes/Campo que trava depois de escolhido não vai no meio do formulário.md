---
tags: [tipo/atomica, camada/padrao, design]
criado: 2026-08-30
---

# Campo que trava depois de escolhido não vai no meio do formulário

> Num formulário de quinze campos, quem preenche entra em modo de despachar: lê
> o rótulo, responde, desce. É o pior estado mental possível para uma escolha
> que não vai poder ser desfeita.

## A regra

Campo cuja escolha **custa caro depois** — endereço público, identificador que
vai circular, moeda da conta, tipo de pessoa jurídica — sai da fila e vira o
primeiro bloco, sozinho, com o custo escrito ao lado:

- **Sozinho no próprio cartão**, para não ser lido como parte de uma sequência.
- **O custo dito antes**, não no erro: "depois de escolhido, só dá para trocar
  a cada 30 dias" acima do campo, e não uma mensagem quando ela tentar.
- **A prévia do resultado embaixo**, porque a pessoa está decidindo sobre uma
  coisa que ainda não existe. Ver `privello.com/duda` enquanto digita é o que
  transforma "escolha um @" em uma decisão concreta.
- **Conferência enquanto digita**, quando a escolha depende do servidor.
  Descobrir que o @ é de outra pessoa depois de quinze campos é perder o
  formulário inteiro para trocar uma palavra. Meio segundo de silêncio antes de
  perguntar, e a resposta carrega o valor que ela descreve — senão o veredito da
  palavra anterior fica colado num campo que já mudou.

## Por que

O resto do formulário é editável para sempre: nome errado se conserta, preço se
atualiza, texto se reescreve. O campo travado é de outra natureza, e tratá-lo
como vizinho dos outros é o que faz a pessoa escolher no automático — e
descobrir o peso da escolha só quando quiser mudar.

## Conexões
- Princípio: [[Identificador que já circulou não é mais seu para mudar]] — o que
  torna o campo caro é ele virar endereço na mão de terceiros.
- Irmã: [[React reseta o formulário ao fim de uma Server Action]] ·
  [[Tela que abre vazia tem que ensinar, tela que abre cheia não]]
- Visto em: [[Privello]]
- Mapa: [[Design]]
