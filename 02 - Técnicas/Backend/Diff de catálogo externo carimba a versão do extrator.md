---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-25
---

# Diff de catálogo externo carimba a versão do extrator

> Como transformar "o fornecedor mudou o catálogo" em changelog automático sem publicar
> mudança que foi sua: o snapshot guarda o número da pipeline que o produziu, e lados de
> pipeline diferente não se comparam.

## O problema

Fonte externa que você consome (catálogo de jogo, tabela de preço, API de terceiro)
muda sem avisar e sem changelog. Você já guarda um snapshot versionado — então o diff
entre o snapshot em disco e o download de agora é o changelog que ninguém escreveu.

Só que o snapshot não é a fonte: é a fonte **normalizada pelo seu ingestor**. Trocar o
ingestor entre duas fotos injeta diferenças que não aconteceram lá fora. Ver
[[Diferença entre duas leituras só fala do mundo se o instrumento não mudou]].

## A solução

Uma constante inteira no motor do diff, gravada em todo snapshot:

```ts
/** Sobe UM a cada mudança na ingestão que altere o valor de qualquer campo
 *  comparado aqui: campo novo, default diferente, escala convertida. */
export const PIPELINE = 2;

export function podeComparar(antes: Snapshot, depois: Snapshot): string | null {
  const pa = antes.pipeline ?? 1;
  const pb = depois.pipeline ?? 1;
  if (pa !== pb) return `ingestão mudou (pipeline ${pa} contra ${pb})`;
  return null;
}
```

Devolver o **motivo** e não um booleano é de propósito: obriga quem chama a olhar por
que não deu, e o motivo vai pro log — "pulei esta passada" é informação, "false" não é.

A comparação roda na única janela em que os dois lados existem: **antes** de sobrescrever
o snapshot. Depois do `writeFile` o passado acabou, e patch não registrado ali é patch
perdido pra sempre. E falhar no diff não pode derrubar a ingestão — o catálogo chegar
atualizado vale mais que a anotação sobre ele.

## O que mais vale lembrar

- **Compare só campo cru.** Nada de derivada sua no diff: derivada que muda é mudança
  sua. A exceção que vale é a derivada que é DEFINIÇÃO e não modelo (soma de
  chance x quantidade x preço), calculada com a mesma fórmula dos dois lados — sem ela
  o diário fica correto e ilegível, porque a linha que decide não está em campo nenhum,
  está na soma deles. Ver [[Ordene pela grandeza que decide, não pela que impressiona]].
- **Ordene por razão, não por diferença**, e mostre a consequência acima da causa: 493
  para 38 de ouro conta mais que 8.000 para 8.600 de XP.
- **Teto por entrada, com o corte CONTADO na tela.** O arquivo é versionado; sem teto,
  uma passada ruim vira um commit de dezenas de MB. Corte silencioso lê como "isto é
  tudo".
- **Date pelo relógio da fonte** (`Last-Modified`), nunca pelo da máquina que rodou —
  ver [[Produtividade se mede pela hora do registro, não pela data do fato]].
- **Rotina, não memória.** O ingestor manual só registra o que você lembrar de rodar. Um
  cron (GitHub Actions a cada 6h, casado com o `max-age` da CDN da fonte) que commita
  quando mudou resolve isso — e de brinde o snapshot de fallback para de envelhecer.
- Na tela, compare o último registro com o carimbo ao vivo da fonte: rotina quebrada
  deixa a página **exatamente igual** — completa, datada e velha.

## Conexões
- Princípio: [[Diferença entre duas leituras só fala do mundo se o instrumento não mudou]]
- Irmã: [[Campo que a normalização não copia vira número errado, não erro]] ·
  [[Sonda que falhou não é sinal de que mudou]] ·
  [[Número de regra alheia se lê da fonte, não se congela em constante]]
- Visto em: [[piwdex2]]
- Mapa: [[Backend]]
