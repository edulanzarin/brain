---
tags: [tipo/atomica, camada/padrao, design]
criado: 2026-09-05
---

# Pergunta fechada se responde com a lista inteira, não com o que sobrou

> Um grupo de etiquetas que desenha só o que a pessoa marcou responde por
> **ausência** — e ausência não distingue "não aceito" de "não preenchi". Para
> pergunta aberta ("que idiomas fala?") tanto faz. Para pergunta fechada ("aceita
> cartão?") é a diferença entre sair de casa e não sair.

## O problema

A ficha do perfil monta os blocos do mesmo jeito: pega as etiquetas do grupo e
desenha uma a uma.

```tsx
{itens.map((i) => <Etiqueta key={i}>{i}</Etiqueta>)}
```

Funciona para serviços, idiomas e atendimento, porque ali a pergunta de quem lê é
aberta: ela quer saber o que existe, e o que não está na lista simplesmente não é
oferecido nem prometido.

Quebra em pagamento. Quem vai encontrar alguém precisa saber se leva dinheiro no
bolso, e "cartão" fora da lista tem duas leituras opostas — ela não aceita, ou ela
não respondeu. As duas mandam a pessoa fazer coisas diferentes, e a tela não
separa.

## A forma

O grupo recebe também o **catálogo inteiro** daquele assunto, e o que ela não
marcou aparece riscado e apagado:

```tsx
const marcadas = new Set(itens);
const lista = todas?.length ? todas : itens;

lista.map((i) => <Etiqueta key={i} recusada={!marcadas.has(i)}>{i}</Etiqueta>)
```

Duas condições, e as duas importam:

- **Só nos grupos de pergunta fechada.** Riscar "espanhol" num perfil que não fala
  espanhol seria o sistema afirmando uma recusa que ninguém fez. Nenhum perfil
  precisa negar idioma.
- **Só se ela marcou alguma.** Com o grupo em branco, riscar tudo afirma que ela
  não aceita forma nenhuma de pagamento — que é exatamente a leitura errada de um
  campo não preenchido. Aí o bloco continua sumindo, como antes.

A segunda condição é o que impede a solução de virar o mesmo erro invertido.

## O que mais vale lembrar

O gatilho para procurar esse defeito é a forma da pergunta de quem lê, não a
forma do dado: no banco os dois casos são a mesma tabela de etiqueta. Sempre que
a pergunta for de sim ou não, a lista precisa carregar o "não" — senão o "não"
está sendo dito pelo silêncio, que é o mesmo canal do "não sei".

## Visto em

No Privello, "formas de pagamento" na ficha do perfil. É o único grupo dos seis
que mostra o que ela não marcou.

## Conexões
- Princípio: [[Todo estado da tela tem visual]] — "recusado" era um estado sem
  desenho, e estado sem desenho vira indistinguível da ausência de dado.
- Irmã: [[Sinal booleano da fonte não ocupa o lugar de uma escala]]
- Irmã: [[Tela que manda comparar duas coisas mostra as duas]]
- Visto em: [[Privello]]
- Mapa: [[Design]]
