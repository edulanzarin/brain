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

## Conexões
- Princípio: [[A variante de um controle muda a intenção, não o tamanho]]
- Irmã: [[Trocar a fonte muda a largura, não só o desenho da letra]]
- Visto em: [[piwdex]]
- Mapa: [[Design]]
