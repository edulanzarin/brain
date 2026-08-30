---
tags: [tipo/atomica, camada/padrao, armadilha, design]
criado: 2026-08-30
---

# Margem negativa em item de flex centralizado vale metade

> `align-items: center` centraliza a caixa DE MARGEM. Encolher essa caixa com
> `margin-top: -64px` sobe o elemento 32px, não 64 — e o restante depende da
> altura do vizinho mais alto da linha.

## O problema

O retrato do perfil devia pousar metade sobre a capa. A receita óbvia:

```html
<div class="flex items-center gap-5">
  <div class="-mt-16">…retrato de 144px…</div>
  <div>…nome, @, cidade, nota…</div>
</div>
```

Encostava uns dez pixels e pronto. A conta explica: a linha tem a altura do
bloco do nome (146px), a caixa de margem do retrato passa a ter 144 − 64 = 80,
e centralizar 80 em 146 põe a borda de cima em (146−80)/2 − 64 = −31. Sem
margem nenhuma ela estaria em +1. Ou seja, os 64 viraram 32.

O pior nem é a metade: é o **146**. Nome que quebra em duas linhas, perfil sem
nota, um selo a mais — qualquer coisa que mude a altura do bloco vizinho move o
retrato. O ajuste "no olho" funciona no perfil em que foi ajustado.

## A solução

Tirar o elemento do alinhamento da linha. Preso ao topo, a margem é exatamente
o que ela diz, e nada do vizinho entra na conta:

```css
.pv-retrato-na-capa {
    align-self: flex-start;
    margin-top: calc(var(--retrato) / -2);
}
```

Duas condições para a conta fechar:

1. **O tamanho vive num token só.** Metade de 144 escrita à mão em outro
   arquivo é a segunda fonte da verdade que desalinha na primeira vez que
   alguém mudar o retrato.
2. **Nada de respiro entre a capa e a linha.** O `gap` da coluna que separa as
   duas entra inteiro na sobreposição. Se houver gap, ou ele some, ou vira
   parcela do `calc` — e parcela invisível é a que ninguém lembra de ajustar.

## Como isso aparece

Não dá erro, não dá aviso, e o número que você escreveu está lá no inspetor.
Só o resultado é outro. O teste que denuncia é mudar o conteúdo do VIZINHO —
se o elemento se move, o alinhamento está no meio da conta.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]] — a
  sobreposição precisa ser fato da estrutura, não resultado que emerge da
  altura de quem está ao lado.
- Irmã: [[Altura 100% em item de grid de linha automática volta ao tamanho intrínseco]] ·
  [[Token semântico em vez de valor literal]]
- Visto em: [[Privello]]
- Mapa: [[Design]] · [[Frontend]]
