---
tags: [tipo/atomica, camada/principio, design]
criado: 2026-07-20
---

# Hierarquia por superfície, não por borda

> Profundidade se constrói com níveis de fundo. Borda é o último recurso, não o
> primeiro.

## A regra

Poucos níveis de superfície, nomeados por profundidade e não por cor: `--page` (o
fundo de tudo), `--surface` (o card em cima da página), `--surface-2` (o que está
dentro do card). Três níveis dão conta de quase toda tela; a partir do quarto, o
problema é o layout, não a paleta.

Borda entra só quando o contraste de superfície não basta — e aí é fio de cabelo
(`--hairline`, rgba com alpha baixo), não traço de 1px sólido em cinza médio.

## Por que

Card com borda em todo lado vira gaiola: a tela ganha uma grade de linhas que compete
com o conteúdo pela atenção. Superfície separa sem desenhar nada — o olho lê "isso
está em cima daquilo" sem que exista traço nenhum.

E funciona nos dois temas. No claro a hierarquia sobe (superfície mais clara que a
página); no escuro sobe também (superfície mais clara que o fundo). A regra é a mesma,
os valores é que trocam — por isso cada nível é token
([[Token semântico em vez de valor literal]]), não hex.

Borda com alpha, em vez de cor fixa, é o que faz o fio funcionar em cima de qualquer
superfície: ele escurece ou clareia o que estiver embaixo, em vez de brigar.

## Conexões
- Depende de: [[Token semântico em vez de valor literal]]
- Padrão que aplica: [[Sistema de cores e tema do dashboard]] · [[Padrões de componentes de dashboard]]
- Irmã: [[Container tem largura máxima e respiro constante]]
- Mapa: [[Base]] · [[Design]]
