---
tags: [tipo/atomica, camada/padrao, design, dev/frontend, armadilha]
criado: 2026-08-18
---

# Trocar arte por ícone de linha exige recalibrar tamanho, não só trocar o componente

> Ícone **desenhado** (pixel art, glifo cheio) e ícone **de linha** (lucide, Feather)
> não vivem na mesma escala. Trocar a implementação sem trocar os tamanhos entrega
> uma tela pior do que a que existia.

## O quê

Um glifo pixel de 8×8 é legível a 8–12px: cada pixel é significativo e o desenho é
maciço. Um ícone de linha nesse tamanho vira borrão — o traço de 2px sobre um
`viewBox` de 24 colapsa, e o antialiasing come a forma.

Ao migrar, o piso é **14px**, com uma escala por contexto:

| Contexto | Tamanho |
|---|---|
| inline ao lado de texto pequeno (14–15px) | 14 |
| inline ao lado de texto de corpo (16px) | 16 |
| sozinho dentro de um botão de 40px | 18 |
| cabeçalho de card/painel | 18 |

Duas armadilhas que só aparecem no meio da migração:

- **Não é `sed`.** A prop `size` costuma ser compartilhada com componentes que
  mostram **arte de verdade** (sprite do jogo, avatar, thumbnail). Essa arte não
  muda de tamanho junto — subir tudo com regex quebra a arte para consertar o ícone.
- **O ícone sem `size` explícito** cai no default do wrapper. Se o default antigo
  era 12 (calibrado para o glifo pixel), toda chamada sem `size` fica errada e não
  aparece em nenhuma busca por `size={N}`. Corrija o default também.

Manter a **mesma API** (`size`/`className`) no wrapper é o que torna a troca barata:
o componente muda por dentro e as dezenas de arquivos que consomem não são tocados.

## Por que importa

No [[piwdex]] a migração dos grids ASCII para lucide manteve a API e não quebrou
nada — mas ~350 chamadas usavam 8–13px, herdados da era do pixel. A troca compilava,
passava no build e ficava **feia**: o menu inteiro virou um borrão cinza. O trabalho
real não foi trocar o componente (uma tarde), foi recalibrar os tamanhos e as
larguras de slot em volta deles.

Vale para a direção contrária também: adotar arte cheia onde havia linha permite
descer de tamanho, e ninguém desce — a tela fica com ícones grandes demais.

## Quando o piso não cabe, troque de CANAL — não encolha o glifo

O piso de 14px é uma restrição, e restrição serve pra reprovar desenho. Uma peça
pequena demais pra caber um ícone legível não pede um ícone menor: pede outra
forma de dizer a mesma coisa.

Caso concreto: numa tier list, cada pokémon é um tile de 76px, e faltava dizer o
TIPO dele — que é a informação que decide o uso. A primeira tentativa copiou o
medalhão do card grande, com o glifo do tipo dentro. Reprovou por duas contas de
escala, e as duas se descobrem medindo:

1. o glifo teria de sair a **12px**, abaixo do piso;
2. dois discos (bitipo) somam ~34px numa peça de 76 — **quase metade da largura**,
   e viram a coisa mais clara do tile, disputando com o sprite a atenção que o
   sprite existe pra receber. O mesmo medalhão no card grande ocupa 18%.

O que entrou foi uma **faixa de cor de 3px** na costura, dividida ao meio no
bitipo. Custa 3px de altura e nenhuma largura, e diz o tipo pela cor — que é o
canal que quem lê tier list já tem calibrado. A palavra fica no `title`, e a faixa
sai do leitor de tela por `aria-hidden`, porque o `title` do botão já diz os dois
tipos por extenso.

Glifo, cor, posição e texto são canais diferentes para o mesmo fato. Quando o
tamanho reprova um, o próximo passo é o canal seguinte — encolher o glifo abaixo
do piso é escolher a versão ilegível do canal que não cabia.

## Trocar de biblioteca quase nunca é a resposta; trocar de REGISTRO é

A queixa costuma chegar como *"esse pacote de ícones não combina"*, e o reflexo é
procurar outro pacote. Quase sempre o que não combina não é o desenho — é o
**registro**: traço fino, neutro, de ferramenta, num produto que pede massa.

O recorte que resolve é por FUNÇÃO, e ele se descobre inventariando:

| grupo | o que fazer |
|---|---|
| **chrome** (chevron, busca, fechar, grade, ordenar) | deixar na biblioteca |
| **domínio** (o que fala do assunto: stat, moeda, item, tipo) | desenhar |

Chrome não ganha nada com desenho — ninguém olha uma seta e sente falta de
personalidade, e desenhar quinze à mão devolve as mesmas quinze setas. Domínio
ganha tudo, porque é o que aparece seis vezes por card e sessenta cards por tela.

**E o glifo de domínio se desenha CHEIO.** Contorno de 2px a 14px perde metade da
forma pro antisserrilhado e o que sobra é cinza; massa se lê. O aviso que vem
junto é o mesmo do resto desta nota ao contrário: cheio pesa mais que traço no
mesmo tamanho, então a troca pede reconferir no tamanho de uso.

Mantendo a API do wrapper (`{ size, className }` + `currentColor`), a troca não
toca nenhuma das centenas de chamadas — e as classes de cor que cada tela passa
continuam valendo.

## Conexões
- Princípio: [[A variante de um controle muda a intenção, não o tamanho]]
- Irmã: [[Trocar a fonte muda a largura, não só o desenho da letra]] ·
  [[Arte de ícone se julga no tamanho de uso, e o acento é a massa]]
- Visto em: [[piwdex]] · [[piwdex2]]
- Mapa: [[Design]]
