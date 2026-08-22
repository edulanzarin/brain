---
tags: [tipo/atomica, camada/padrao, design, dev/frontend]
criado: 2026-08-21
---

# Fila de campos alinha por altura fixa de controle, não por items-end

> Uma barra de filtros só fica reta quando **todo controle tem a mesma altura** e
> **a célula do campo reserva a linha do rótulo mesmo sem rótulo**. `items-end`
> não alinha nada: ele só disfarça alturas diferentes encostando os pés — e
> qualquer controle sem rótulo continua 20px acima dos vizinhos.

## O sintoma

No [[piwdex2]], a barra de cenário da Hunt tinha três alturas na mesma fila:
campo de texto e select em 40px (a casca `.field` do projeto), segmentado em
32px, e um switch **sem casca nenhuma**, boiando no meio. O Eduardo apontou de
relance: "um input maior que o outro, aquele switch 50% mais baixo, o select de
tipo mais alto que o outro".

A causa não era descuido de uma tela. Era que **cada tela montava a própria
célula na mão**:

```tsx
<div>
  <FieldLabel className="mb-1">Área</FieldLabel>
  <Select ... />
</div>
<Switch checked={vip} label="VIP" />   {/* sem rótulo em cima: sobe 20px */}
```

Enquanto a célula é improvisada por tela, basta um controle com altura própria
pra desalinhar a fila inteira — e o conserto vira margem corretiva por instância,
que é a mesma doença de [[Primitiva de botão fecha o tamanho e abre só a variante]].

## A técnica: uma célula, duas regras

```tsx
export function Field({ label, icon, hint, className, children }) {
  return (
    <div className={cn("flex min-w-0 flex-col gap-1", className)}>
      <FieldLabel className="flex h-5 items-center gap-1.5">
        {label ? <>{icon}<span className="truncate leading-[1.5]">{label}</span></>
               : <span aria-hidden="true">&nbsp;</span>}   {/* reserva a linha */}
      </FieldLabel>
      {children}
      {hint ? <span className="text-[12px] text-text-mute">{hint}</span> : null}
    </div>
  );
}
```

1. **Todo controle veste a mesma casca.** O que já tinha altura (input, select,
   combobox) não muda; o que não tinha ganha: o segmentado passa a ter a altura
   da casca, e switch e checkbox ganham um `boxed` que os veste com ela. Um
   controle de 32px numa fila de 40 é exatamente os 8px que o olho pega.
2. **O rótulo ocupa lugar mesmo quando não existe.** Célula sem rótulo renderiza
   a linha vazia. Sem isso o controle vizinho sobe a altura do rótulo — que é o
   caso do switch, que carrega o próprio texto ao lado e por isso não tem label
   em cima.

Com as duas, a fila alinha sozinha: nada de `items-end`, nada de `mt-` corretivo
em tela nenhuma.

## A armadilha que veio junto: truncar corta o acento

O rótulo trunca (`overflow: hidden`) pra caber na coluna. Mas truncar esconde nos
**dois eixos**: numa linha de 11px com `leading-none`, o acento fica fora da
caixa e some — "ÁREA" virava "AREA", "ESPÉCIE" virava "ESPECIE". O texto parecia
sem acento e o bug parecia de fonte.

A altura fixa da célula não é o problema (é ela que alinha a fila); o problema é
a **linha apertada dentro da parte que trunca**. `leading-[1.5]` no span truncado
resolve sem desalinhar nada.

## Conexões
- Princípio: [[Escala fechada em vez de valor solto]] ·
  [[A variante de um controle muda a intenção, não o tamanho]]
- Irmã: [[Primitiva de botão fecha o tamanho e abre só a variante]] ·
  [[Controles de filtro do dashboard]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]]
