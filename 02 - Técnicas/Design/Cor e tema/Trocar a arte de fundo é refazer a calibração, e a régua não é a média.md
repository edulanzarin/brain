---
tags: [tipo/atomica, camada/padrao, design, dev/frontend, armadilha]
criado: 2026-08-24
---

# Trocar a arte de fundo é refazer a calibração, e a régua não é a média

> Fundo de arte não é um arquivo que se substitui: é um arquivo com números
> calibrados em cima dele. Trocar a arte sem remedir mantém números que foram
> derivados de outra imagem — e o pior caso não é o fundo ficar feio, é ele ficar
> **plano**, com o desenho ainda lá e ninguém conseguindo vê-lo.

## Duas réguas, e nenhuma das duas é a luminância média

### O scrim se calibra pelo PERFIL VERTICAL

O scrim é um gradiente: ele fecha pouco no topo e muito embaixo. A arte também
não é uniforme. Logo, comparar um gradiente com um escalar (a média da imagem)
não responde a pergunta que interessa, que é *quanto sobra da arte em cada faixa
da tela*.

Medindo por célula — média e desvio por faixa horizontal — a decisão aparece
sozinha. Num caso real, três recalibrações seguidas: `70% → 40% → 18%` de scrim
no topo. E a terceira arte, a que pediu o scrim **mais aberto**, era a mais
**clara** das três: média 77 na fonte contra 19 da anterior.

Parece contradição e não é. A média subiu porque o RODAPÉ da arte é claro; a
faixa de cima, onde o herói mora e onde o scrim é mais leve, continuava em 16-25.
Aplicar o scrim antigo nela achatava a cena inteira pra 14-22 de luminância em
toda a altura — um retângulo escuro onde dá pra adivinhar que existe um desenho.

```
por faixa, depois de assar:
  topo    17  16  24  25  21  24     <- scrim leve aqui
  meio    38  31  37  44  38  42
  rodapé  43  49  56  52  45  70     <- scrim pesado aqui
```

### O desfoque se decide pelo GÊNERO da arte

Regra que parecia estável e caiu: *a imagem não leva blur, quem borra é o vidro
dos painéis*. Ela vale enquanto o fundo for geometria, textura ou luz pontual —
não há nada ali que o olho tente **ler**.

Ilustração é outro caso. Ela tem personagem, rosto, contorno. Nítida atrás de um
painel de vidro, ela não lê como ambiente: lê como conteúdo que alguém cobriu, e
o olho volta pra ela. Um desfoque leve, assado no arquivo, resolve — e assado, e
não em `filter`, porque desfoque numa camada `fixed` de tela cheia é a conta mais
cara que se pede ao compositor, e ela se paga a cada scroll.

Raio como **fração da largura**, nunca em pixel fixo: os dois perfis (o de 2560 e
o de 1280) precisam sair com o mesmo desfoque aparente.

## O que mais vale lembrar

**Tratamento que se derrama fica.** Junto com a pixelização removida do script
sobreviveu um `image-rendering: pixelated` no CSS — mandando o navegador escalar
com vizinho-mais-próximo uma imagem que agora nascia suavizada. Ninguém viu
porque o defeito só aparece em janela mais larga que o arquivo. Ao tirar um
tratamento da produção da arte, procure a linha que o acompanhava no consumo.

**A fonte se acha por padrão, não por extensão cravada.** Um `FONTE =
assets/wallpaper.jpg` fixo devolve "faltou a fonte" quando a arte nova chega em
png, com o arquivo ali do lado. Trocar o wallpaper tem que ser trocar o arquivo.

## Conexões
- Princípio: [[A régua sai da distribuição, não dos extremos]]
- Irmã: [[Sobre arte de fundo, a chrome também tem piso de opacidade]] ·
  [[Vidro flutuante precisa de superfície mais opaca que a chrome]] ·
  [[Animação de enfeite escolhe a propriedade pelo custo, não pelo efeito]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]]
