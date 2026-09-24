---
tags: [tipo/atomica, camada/principio, seguranca]
criado: 2026-09-24
---

# Afirmação que chega de fora só vale com um código que a casa emitiu antes

> "Este vídeo sou eu agora", "este grupo é meu", "este domínio é meu": toda
> afirmação que entra no sistema por um canal que ele não controla é só uma
> afirmação. O que a transforma em prova é a casa ter emitido um código ANTES, por
> um canal que só o dono enxerga, e exigir que ele volte junto.

## A regra

1. O sistema gera um valor que ninguém tinha como adivinhar.
2. Entrega esse valor só a quem já provou quem é (a sessão logada, o painel).
3. Aceita a afirmação externa apenas se ela trouxer o valor de volta.

Sem o passo 2, o código não prova nada: qualquer um que chegue à mesma tela
pública copia. Sem o passo 3, a primeira afirmação plausível vence, e a
plausível é justamente a que o atacante monta.

## Por que

O canal externo é aberto por natureza. Um bot do Telegram pode ser posto em
qualquer grupo por qualquer pessoa que saiba o nome dele; um vídeo pode ter sido
gravado há três anos; um CNAME pode ser apontado por quem nunca foi dono do
domínio. Confiar no primeiro sinal que chega ("o bot virou administrador em um
grupo, então esse é o grupo") entrega a decisão a quem chegou primeiro.

O defeito não dá erro: o fluxo funciona perfeitamente, só que para a pessoa
errada. No caso do bot, os convites pagos passariam a apontar para o grupo de
quem sequestrou o vínculo, e o comprador pagaria para entrar no lugar errado.

## Onde aparece

- Verificação de pessoa por vídeo com código sorteado:
  [[Artefato prova existência; só um desafio prova o momento]] (ali o código ainda
  tem prazo, porque o que se prova é o momento).
- Vínculo de grupo e de dono num bot do Telegram:
  [[Bot que qualquer um pode pôr num grupo se vincula a ele por código]].
- Fora daqui, a mesma forma: o registro TXT que o provedor de DNS pede, o `state`
  do OAuth, o código que o banco manda antes da transferência.

## Conexões
- Técnica que aplica: [[Bot que qualquer um pode pôr num grupo se vincula a ele por código]] ·
  [[Artefato prova existência; só um desafio prova o momento]]
- Irmã: [[A assinatura autentica o dado, não quem o trouxe]] ·
  [[Permissão se valida no servidor, não na interface]]
- Visto em: [[Privello]] · [[telebot]]
- Mapa: [[Base]]
