---
tags: [tipo/atomica, camada/padrao, dev/backend, design]
criado: 2026-09-05
---

# O que a página afirma sai do mesmo dado que ela desenha

> Uma página de listagem fala por três canais: o que a pessoa lê, o que o robô
> lê no JSON-LD, e o texto que descreve o conjunto. Escritos separado, os três
> divergem — e o canal que ninguém relê é o que envelhece mentindo.

## O caso do texto

Uma vitrine de cidade com um H1, uma grade de fotos e a frase "177 anúncios no
ar" é uma página rasa: compete por "acompanhantes em Blumenau" contra páginas
que têm parágrafo, e perde. A saída óbvia é escrever um parágrafo sobre a
cidade — clima, ponto turístico, "no coração do vale".

Não. Esse texto é sobre um assunto que o sistema **não conhece**: ninguém vai
reler duzentos parágrafos, e nenhum deles muda quando o banco muda.

O que o sistema conhece é o próprio conjunto. O parágrafo se **mede**:

```
13 anúncios no ar em Blumenau — SC, todos com documento conferido antes da
publicação. A hora vai de R$ 150 a R$ 450. 9 têm vídeo de comparação
conferido, que qualquer pessoa pode assistir no perfil.
Bairros com anúncio no ar: Centro, Fortaleza, Garcia, Itoupava...
```

Uma agregação — mínimo, máximo, contagens, valores distintos — e uma função que
monta frases. Regra dura: **frase sem dado que a sustente some, em vez de sair
zerada**. "0 no ar agora" e "de R$ 0 a R$ 0" são piores que silêncio, porque
afirmam vazio onde houve uma medida que não deu em nada.

## O caso da marcação

JSON-LD é o mesmo problema com outro leitor. A tentação é declarar tudo que o
vocabulário aceita — nota, preço, horário, disponibilidade — porque cada campo
parece um ponto a mais. Marcação que afirma o que a tela não mostra não é
otimização: é o caminho conhecido para uma ação manual.

Então cada campo tem par visível, e as ausências são deliberadas:

- A nota só sai **com avaliação de verdade**. Nota inventada, ou "5,0 de 0
  avaliações", é o caso clássico de penalidade.
- Nada sai na **prévia** de um registro que ainda não foi publicado: marcação de
  página que responde 404 amanhã é sinal que o robô cobra do site inteiro.
- Campo que o vocabulário não tem não se inventa. `PostalAddress` não tem
  bairro, e escrever `addressNeighborhood` é declarar propriedade inexistente —
  ignorada no melhor caso, e no pior a assinatura de marcação gerada sem ler a
  especificação.

## O que mais vale lembrar

Os construtores ficam num módulo só, e não espalhados nas páginas, por um
motivo específico deste tipo de saída: é a única forma de garantir, olhando um
arquivo, que **todo campo declarado tem origem no mesmo dado que a tela usa**.
Espalhado, ninguém consegue afirmar isso — e ninguém percebe quando deixa de
ser verdade, porque a tela continua certa.

## Visto em

No Privello, o parágrafo da vitrine de cidade e os nós de `ProfilePage`,
`Person` e `Service` do perfil.

## Conexões
- Princípio: [[Dado escrito por dois caminhos precisa de uma regra só, fora dos dois]]
- Irmã: [[A superfície indexável sai da mesma consulta que o conteúdo]] — lá é a
  rota que sai da consulta do conteúdo; aqui é o que a página afirma sobre ele.
- Irmã: [[Filtro é ferramenta, recorte é página]]
- Visto em: [[Privello]]
- Mapa: [[Backend]]
