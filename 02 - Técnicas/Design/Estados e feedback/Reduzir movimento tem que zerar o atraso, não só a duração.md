---
tags: [tipo/atomica, camada/padrao, design, dev/frontend, armadilha]
criado: 2026-08-22
---

# Reduzir movimento tem que zerar o atraso, não só a duração

> O reset padrão de `prefers-reduced-motion` zera duração e repetição e deixa
> `animation-delay` intacto. Com entrada em cascata, isso vira tela em branco pra
> exatamente quem pediu pra não ver movimento.

## O reset que quase todo mundo copia

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

Ele resolve o caso comum e falha no caso escalonado.

## Por que falha

Entrada em cascata precisa de duas coisas: `both` no preenchimento (pra o elemento
já nascer no quadro inicial, invisível) e um atraso por posição:

```css
.anim-in { animation: pix-rise 420ms ease both; animation-delay: var(--d, 0ms); }
```

Com a duração zerada e o atraso preservado, o elemento fica preso no keyframe
`from` — `opacity: 0` — durante todo o atraso, e só então salta pro estado final.
Num grid com teto de 16 passos de 28ms, ou num herói com quatro blocos até 320ms,
são centenas de milissegundos de conteúdo ausente. E não é degradação elegante: o
conteúdo **não está lá**.

## A correção

```css
    animation-delay: 0ms !important;
```

Uma linha, no mesmo bloco. Vale pra `--d` (cascata de herói) e `--i` (cascata de
grid), que são a mesma técnica em escalas diferentes.

## Como conferir sem depender do olho

O navegador headless aceita a preferência como configuração de contexto, então isso
entra em teste de verdade:

```js
const p = await b.newPage({ reducedMotion: "reduce" });
await p.goto(url, { waitUntil: "domcontentloaded" });
await p.screenshot({ path: "reduzido.png" });   // sem waitForTimeout de propósito
```

O `waitForTimeout` generoso é o que esconde o defeito: espere 900ms e a tela já
apareceu, atraso ou não. Tire a espera e o buraco aparece.

## A exceção que continua valendo

Indicador de **progresso** não congela junto. Quem liga a preferência quer menos
movimento, não uma tela com cara de travada — o indicador perde o deslocamento e
mantém o pulso. Enfeite (flutuar, brilhar) não entra nessa exceção.

## Conexões
- Princípio: [[Todo estado da tela tem visual]]
- Irmã: [[Animação de enfeite escolhe a propriedade pelo custo, não pelo efeito]] ·
  [[Faixa de topo de ferramenta é chegada, não rótulo]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]]
