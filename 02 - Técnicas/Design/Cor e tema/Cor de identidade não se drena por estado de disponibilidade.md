---
tags: [tipo/atomica, camada/padrao, design, armadilha]
criado: 2026-09-02
---

# Cor de identidade não se drena por estado de disponibilidade

> Cada módulo do sistema ganhou cor própria — e o cartão dele ficava cinza
> enquanto as telas não existissem. Como NENHUM módulo estava pronto ainda, o
> launcher inteiro nasceu cinza: o sistema de cor recém-construído era invisível
> exatamente no período em que ele seria julgado.

## O problema

São dois eixos diferentes, e é fácil colapsá-los num só:

| Eixo | Pergunta que responde | Como se mostra |
|---|---|---|
| **Identidade** | que coisa é esta? | cor própria, ícone próprio, nome |
| **Disponibilidade** | posso usar agora? | selo, interação, opacidade |

Drenar a cor para dizer "ainda não dá para entrar" usa o canal da identidade
para falar de estado. O efeito colateral é o pior possível numa transição: no
começo, quando NADA está pronto, a tela inteira perde a identidade de todo mundo
ao mesmo tempo — e parece desligada, não em construção.

## A regra

**A cor pertence à coisa, não à permissão de usá-la.** O que muda com o estado é
o selo, o cursor, a elevação no apontador e o texto de rodapé.

A exceção é estreita e tem motivo próprio: **"sem acesso" pode ir a cinza**,
porque ali a mensagem não é "isto ainda não existe", é "isto não é seu". Tirar a
identidade é justamente o recado.

```tsx
// a cor segue a entidade; só a ausência de permissão a apaga
const colorido = estado !== "sem-acesso";
const interativo = estado === "disponivel";
```

## O que mais vale lembrar

- O sintoma que denuncia: **você constrói um sistema de cor e não consegue vê-lo
  na própria tela**. Se a cor só aparece num estado que quase nada alcança, ela
  está pendurada no eixo errado.
- Vale para além de cor: ícone próprio, avatar e logotipo são identidade pela
  mesma razão. Substituí-los por um genérico enquanto algo está indisponível
  apaga a única pista de qual coisa é aquela.
- Estado que some com a identidade também atrapalha a busca visual: numa grade
  de seis, a pessoa procura pelo azul do Fiscal, não pela palavra.

## Conexões
- Princípio: [[Token semântico em vez de valor literal]]
- Irmã: [[Acento da interface é um token separado da cor de dado]] · [[Fato vai em selo, estado vivo vai no retrato]] · [[Estado bloqueado aponta para a chave]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Design]]
