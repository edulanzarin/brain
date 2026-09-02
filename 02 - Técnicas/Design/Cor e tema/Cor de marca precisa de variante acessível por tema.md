---
tags: [tipo/atomica, camada/padrao, design, dev/frontend]
criado: 2026-08-06
---

# Cor de marca precisa de variante acessível por tema

> A cor que o cliente escolhe raramente serve como está: um laranja ou verde vibrante
> reprova contraste quando vira texto. Calcule o contraste e derive uma variante por tema.

## O quê

Ao aplicar uma paleta de marca, a mesma cor cumpre papéis diferentes — preenchimento de
botão, texto de link, tinta de ícone, borda. Cada papel tem um piso de contraste (WCAG
AA: 4.5:1 texto normal, 3:1 UI e texto grande). Um tom saturado costuma passar como
*fill* (com tinta clara por cima) e **reprovar como texto** sobre a superfície.

A regra: **um token de marca por tema, derivado por cálculo, não a cor "crua".**

- Claro: escureça a marca até passar AA como texto sobre a superfície clara (laranja
  `#c2410c` ~5:1, não `#ea580c` ~3.6:1).
- Escuro: clareie a marca até passar sobre a superfície escura (`#fb923c`), com
  `brand-ink` escuro por cima do fill.
- `brand-soft` (tint translúcido) para áreas grandes e estados ativos — legível com
  tinta de texto normal, ao contrário do fill saturado que cansa.

Corolário de gosto: **fundo fica neutro** (brilho colorido ambiente é rejeitado — "fica
feio, tem que ser mais clean") e **fill grande atrapalha a leitura** — balão de mensagem
vira tint claro com texto escuro, não bloco saturado com texto claro.


## O eixo que manda é o PAPEL, não o tema

O título fala em "por tema", mas o tema é só um dos eixos — e o menos essencial.
Num sistema de **tema único**, a divisão continua obrigatória, porque quem obriga
é o papel que a cor cumpre. No [[Navetech Hub]], cada módulo virou **quatro
tokens**:

| token | papel | piso |
|---|---|---|
| `vivo` | a cor de verdade: preenchimento de área grande, aresta | — |
| `tinta` | a variante escurecida, para TEXTO e ícone sobre a superfície | 4.5:1 |
| `chip` | o fundo tingido em que o ícone vive | ~1.05:1 vs superfície |
| `sobre` | o que se lê EM CIMA da cor viva | 4.5:1 |

Quem obrigou foi o **amarelo**. Com um token só, há duas saídas e as duas são
ruins: ou o amarelo é escurecido até passar como texto — e aí ele deixa de ser
amarelo em toda parte, virando marrom —, ou fica cru e a sigla branca em cima
dele reprova. Com os quatro, o cubo é amarelo de verdade e a sigla nele é quase
preta, enquanto o texto do menu usa a variante escura.

O `sobre` é o token que costuma faltar. Um "branco para todos" funciona em cinco
cores de seis e quebra exatamente na clara — e quebra em silêncio, porque a peça
continua desenhando.

E vale a conta em bloco: com sete identidades e quatro papéis, são 28
combinações. Nenhuma delas se confere de cabeça.

## Por que importa

"Não avalie contraste de cabeça — compute." É o mesmo hábito de
[[Validar paleta de gráficos antes de escolher cores]], só que para texto e UI: rode um
cálculo de razão de contraste em cada par (marca/superfície, ink/marca) antes de fechar.
O token semântico faz a troca sair em dois lugares (claro e escuro) sem tocar componente.

## Conexões
- Princípio: [[Token semântico em vez de valor literal]]
- Irmã: [[Validar paleta de gráficos antes de escolher cores]] · [[Sistema de cores e tema do dashboard]]
- Visto em: [[Navehub]] · [[Navetech Hub]]
- Mapa: [[Design]]
