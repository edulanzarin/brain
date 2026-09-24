---
tags: [tipo/atomica, camada/padrao, design, dev/frontend, armadilha]
criado: 2026-09-17
---

# Entrada animada com preenchimento both prende o transform e anula o hover

> A entrada de bloco (sobe e aparece) costuma ser escrita com
> `animation-fill-mode: both`. Depois que ela termina, o `transform` do último
> quadro continua aplicado pela animação, e valor de animação vence qualquer
> regra comum da folha — inclusive o `:hover` que ergue o cartão. O cartão entra
> bonito e nunca mais responde ao apontador.

## Por que acontece

Na cascata do CSS, o que uma animação aplica fica acima das declarações normais
do autor. Com `forwards` ou `both`, o último quadro segue "animando" para sempre:
`transform: none` preso no elemento. O `:hover { transform: translateY(-2px) }`
perde, sem erro nenhum.

## A saída

- **`backwards`, não `both`.** A entrada só precisa do quadro inicial durante o
  atraso (o bloco invisível esperando a vez na cascata). Quando o último quadro é
  o estado natural do elemento — opacidade 1, sem deslocamento —, soltar a
  animação no fim não muda o desenho e devolve o elemento às regras dele.
- **Gestos diferentes em propriedades diferentes.** A entrada anima `transform`;
  o hover mexe em `translate` e o clique em `scale`, que são propriedades
  individuais e se compõem com o `transform` em vez de disputá-lo. (No Tailwind
  v4, `-translate-y-px` e `scale-*` já geram `translate`/`scale`.)

## A cascata sem numerar nada

Irmãos que entram juntos chegam um atrás do outro com o atraso pela posição, na
própria folha:

```css
.entra { animation: surge 460ms var(--curva-saida) backwards; }
.entra:nth-child(2) { animation-delay: calc(var(--passo) * 1); }
/* ... até um teto, para o décimo bloco não esperar meio segundo */
```

A tela não passa índice para ninguém: usar o painel já dá a cascata. E o atraso
precisa zerar em `prefers-reduced-motion` — só encurtar a duração deixa o bloco
invisível durante o atraso intacto.

A mesma separação serve quando o `transform` já é da POSIÇÃO: numa pilha em que
cada item fica em `translateY(i * 12px) scale(...)`, a entrada do item novo anima
`translate`, `rotate` e `scale`, e as duas se somam em vez de uma apagar a outra
([[Na lista que recebe item ao vivo, a chave nova faz a entrada e a transição move o resto]]).

## Conexões
- Princípio: [[Todo estado da tela tem visual]]
- Irmã: [[Animação de enfeite escolhe a propriedade pelo custo, não pelo efeito]] ·
  [[Reduzir movimento tem que zerar o atraso, não só a duração]]
- Visto em: [[Navetech Hub]] · [[telebot]]
- Mapa: [[Design]]
