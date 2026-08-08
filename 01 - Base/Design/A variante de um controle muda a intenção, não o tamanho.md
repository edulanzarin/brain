---
tags: [tipo/atomica, camada/principio, design]
criado: 2026-08-08
---

# A variante de um controle muda a intenção, não o tamanho

> Um botão (ou qualquer controle) tem **um gabarito só** — mesma altura, mesmo
> alvo, mesmo raio. O que muda de uma instância pra outra é a **intenção**:
> primário, neutro, perigo, fantasma. Tamanho e intenção são eixos separados, e
> só a intenção varia por uso.

## A regra

Quando você for desenhar as variações de um controle, separe os eixos:

- **Forma** (altura, padding, raio, tamanho do alvo): **fechada**, um valor só.
  É o que dá o "mesmo tamanho, mesmo tudo".
- **Intenção** (cor/ênfase: primário, neutro, perigo, sucesso): **aberta**, é o
  que a variante escolhe.

Se pra um botão específico você precisa mexer na *altura* ou no *padding*, o
sinal não é "esse botão é especial" — é que a forma vazou pro uso. A resposta é
revisar o gabarito único, não abrir exceção local (mesma lógica de
[[Escala fechada em vez de valor solto]]: exceção local é como o padrão morre,
uma de cada vez).

## Por que

Tela com "botão grande, botão pequeno" quase nunca é decisão de design — é
acúmulo de ajustes pontuais de tamanho que ninguém enxerga isolados, mas que
somados fazem a interface parecer desalinhada e amadora. Ter **um** tamanho tira
essa decisão do calor do momento: "que botão uso aqui?" vira uma pergunta de
*qual intenção*, com 4–5 respostas, não de *qual tamanho*, com infinitas.

O efeito colateral bom é ritmo: controles do mesmo gabarito alinham entre si
mesmo em telas que nunca se falaram.

## Como se garante

Um princípio que depende de disciplina não sobrevive. A forma de fechar a forma
é **não oferecer a alavanca**: o controle mora numa primitiva que expõe
`variant` (intenção) e **não** expõe `size`. Sem o botão de tamanho, o override
por instância deixa de existir por construção — não por convenção. É a diferença
entre "combinamos de não redimensionar" e "não dá pra redimensionar".

Rótulo (chip/tag) é diferente de controle: ele carrega informação, não ação, e
pode legitimamente ter duas escalas (inline vs solto). O que não pode é o
**controle de ação** virar régua.

## Conexões
- Irmã: [[Escala fechada em vez de valor solto]] — aquela fecha os *valores* de
  cada eixo; esta fecha *quais eixos* de um controle variam.
- Vizinha: [[Token semântico em vez de valor literal]] — a intenção é nomeada
  (primário/perigo), não um hex solto.
- Técnica: [[Primitiva de botão fecha o tamanho e abre só a variante]]
- Mapa: [[Base]] · [[Design]]
