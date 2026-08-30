---
tags: [tipo/atomica, camada/padrao, design]
criado: 2026-08-30
---

# Palavra da interface é lida com o dicionário do usuário, não com o seu

> "Endereço" era um termo técnico exato: o caminho do perfil no site. Num
> classificado de acompanhantes, é também o lugar onde ela mora — e o campo
> passou a parecer que estava pedindo isso.

## O problema

O vocabulário do sistema nasce de dentro. `slug`, `endereço`, `identificador`,
`registro`, `conta` — cada um é preciso na cabeça de quem construiu, e a
precisão engana: a palavra parece boa porque descreve bem a coisa. Só que ela
não chega sozinha na tela. Chega junto do resto do domínio, e o domínio já tem
dono para muitas dessas palavras.

Dois jeitos de a colisão custar caro:

- **Colisão com um conceito sensível.** "Seu endereço" acima de um campo, num
  produto onde a pessoa precisa esconder onde mora, assusta antes de explicar.
  Nenhum texto de ajuda desfaz o susto do rótulo.
- **Colisão com outra coisa do próprio sistema.** O cadastro pede "nome" e o
  anúncio pede "nome" de novo, e são coisas diferentes — uma é o nome de
  registro, a outra é o nome público. A mesma palavra em dois campos vira a
  pergunta "isto substitui aquilo?", e quem não perguntar publica o nome errado.

## A correção

**Fale da coisa, não da categoria dela.** No lugar de "o @ é o endereço do seu
perfil", *"o @ é o que vem depois de privello.com/"*. Some a palavra disputada e
sobra a descrição, que não colide com nada porque não é um termo, é uma frase.

**Quando duas coisas do sistema dividem a palavra, o rótulo diz qual é.** "Nome
no anúncio", com a nota *"é outro nome, não o da sua conta"*. E o campo NÃO vem
preenchido com o outro: o padrão que a pessoa aceita sem pensar é onde o engano
vira dado.

**A troca vale no produto inteiro, não na tela onde doeu.** Duas telas usando
palavras diferentes para a mesma coisa é o defeito seguinte.

**No código a palavra técnica fica.** `enderecoDoPerfil()` continua sendo o
nome certo da função — lá não há com o que confundir, e trocar por eufemismo só
piora a leitura de quem mantém.

## Como achar antes de doer

Liste os substantivos dos rótulos e pergunte, para cada um: **neste domínio,
essa palavra já significa outra coisa?** Em produto sensível (saúde, dinheiro,
segurança, adulto), a resposta costuma ser sim mais vezes do que se espera.

## Conexões
- Princípio: [[Nota carrega só o que a pessoa não sabe]] — a nota do campo existe
  para dizer o que a tela sabe e a pessoa não; quando o rótulo mente, nenhuma
  nota conserta.
- Irmã: [[Texto de interface soa a IA pelo ritmo, não pelo assunto]] ·
  [[Traduza o vocabulário do sistema, não o nome próprio]] ·
  [[Ranking de opções não usa o verbo do estado ao vivo]]
- Visto em: [[Privello]]
- Mapa: [[Design]]
