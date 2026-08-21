---
tags: [tipo/atomica, camada/padrao, design, armadilha]
criado: 2026-08-20
---

# A régua de um medidor é percentil, não máximo

> Escalar a barra pelo maior valor do conjunto parece a escolha honesta e é a que
> destrói o sinal: basta um outlier pra empurrar todo mundo pro primeiro terço da
> altura e transformar seis barras diferentes numa faixa chapada.

## O problema

Barra, sparkline, mini-gráfico de perfil — todos precisam de um teto. O reflexo é
`max(conjunto)`, porque é o único número que garante que nada estoura.

Só que o teto não é escolhido pra caber: é escolhido pra **separar**. E a distribuição
real quase nunca é uniforme. Medido num catálogo de 2.604 valores: mediana 65, p98 130,
máximo 255 — e só **0,7% passavam de 150**. Com 255 no teto, 99,3% das barras viviam
abaixo de metade da altura, e a diferença entre 45 e 65 (que é o que o gráfico existia
pra mostrar) virava dois pixels.

O sintoma engana: nada quebra, nada estoura, o gráfico "funciona". Ele só não informa.

## A solução

Teto no **percentil** (p95–p98), e quem passa **satura marcado**:

```tsx
// medido uma vez, no servidor, junto com o resto das derivações
const p98 = todos.sort((a, b) => a - b)[Math.floor(todos.length * 0.98)];
const teto = Math.ceil(p98 / 10) * 10;   // régua redonda se lê melhor que 131

const razao = valor / teto;
<span style={{ height: `${Math.max(8, Math.min(100, razao * 100))}%` }} />
{razao > 1 ? <span className="tampa" /> : null}   // "estourou a régua"
```

Três detalhes que fazem parte da regra, não são enfeite:

- **A marca de saturação é obrigatória.** Sem ela, 140 e 255 desenham a mesma barra cheia
  e o gráfico afirma que são iguais — trocou um erro por outro.
- **Piso de ~8%.** Valor baixíssimo não pode virar barra invisível, que se confunde com
  "não carregou" (ver [[Zero num medidor é estado, não barra vazia]]).
- **O teto é do universo inteiro, não da página filtrada.** Régua que muda com o filtro
  faz o mesmo item ter barra de tamanho diferente em duas telas, e aí a comparação —
  a única coisa que o medidor entrega — passa a mentir.

## O que mais vale lembrar

A pergunta que revela o erro é **"quantos itens ficam acima de metade da barra?"**. Se a
resposta for "quase nenhum", a régua está errada, mesmo que o desenho esteja "correto".

Vale pra qualquer escala derivada de dado, não só barra: eixo de gráfico, tamanho de
bolha, intensidade de heatmap. O outlier legítimo não deve ser escondido — deve ser
**marcado como outlier** em vez de reescrever a régua de todo mundo.

## Visto em

No piwdex2 o card da Pokédex tem uma "espinha" de seis barrinhas cujo único trabalho é
deixar ler o PERFIL da espécie de relance — um Onix é visivelmente um muro, um Electrode
é visivelmente uma flecha. Com o máximo do catálogo (255) no teto, todos os cards
desenhavam a mesma faixa baixa e a espinha não distinguia nada. Com p98 (130) e tampa nos
que estouram, o perfil voltou a aparecer.

## Conexões
- Princípio: [[Todo estado da tela tem visual]]
- Irmã: [[Zero num medidor é estado, não barra vazia]] · [[Validar paleta de gráficos antes de escolher cores]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]]
