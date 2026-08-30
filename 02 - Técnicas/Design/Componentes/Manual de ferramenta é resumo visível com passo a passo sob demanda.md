---
tags: [tipo/atomica, camada/tecnica, design, react]
criado: 2026-08-21
---

# Manual de ferramenta é resumo visível com passo a passo sob demanda

> Uma frase sempre na tela, os seis passos dentro de um `<details>`. Manual aberto por
> padrão empurra a ferramenta pra fora da primeira dobra — que é o único lugar onde ela
> funciona.

Forma concreta do princípio [[Tela que abre vazia tem que ensinar, tela que abre cheia não]].

## O componente

Uma faixa de ~40px, na cor da ferramenta, acima do formulário:

```tsx
<details className="panel group" style={{ borderColor: `color-mix(in oklab, ${cor} 34%, var(--color-line))` }}>
  <summary className="flex cursor-pointer list-none items-center gap-3 px-3 py-2.5
                      [&::-webkit-details-marker]:hidden">
    <span className="pix" style={{ color: cor }}>Como usar</span>
    <span className="flex-1 truncate group-open:whitespace-normal">{resumo}</span>
    <IconChevronDown className="transition-transform group-open:rotate-180" />
  </summary>
  <div className="border-t border-line p-4">{/* passos numerados + ressalvas */}</div>
</details>
```

Três decisões que carregam o padrão:

1. **A barra fechada anuncia o tamanho do manual.** Este ponto foi corrigido em
   ago/2026 — ver "A frase mudou de lugar" logo abaixo. A versão original punha aqui o
   resumo, a frase que diz o que a ferramenta responde.
2. **`<details>`, não estado de React.** Abrir e fechar aqui é **chrome, não dado**: não
   vai pra URL (≠ [[Filtro de lista mora na URL]]), não precisa de efeito, não precisa de
   JS. Consequência prática no Next: o bloco continua **componente de servidor** e o
   manual nasce no HTML — inclusive pro buscador, que é quem traz gente nova. Teclado e
   leitor de tela vêm de graça.
3. **O tint é o da ferramenta**, o mesmo da navegação e da home, pra o manual ser lido
   como parte dela e não como aviso do site.

Detalhes que só aparecem montando: o `summary` inteiro é o alvo do clique (faixa, não
chevron de 16px); o marcador nativo sai com `list-none` **e**
`[&::-webkit-details-marker]:hidden`; e o corpo só ganha animação de abertura no seletor
`details[open] > .anim-abre` — numa classe solta ela rodaria uma vez na carga, com o
painel fechado, ou seja, nunca seria vista.

## A frase mudou de lugar (ago/2026)

Quando [[Faixa de topo de ferramenta é chegada, não rótulo]] entrou, a faixa de topo
passou a carregar a frase que diz o que a ferramenta faz. A barra do manual continuava
carregando **a mesma frase**, dois blocos abaixo. Ler a mesma coisa duas vezes em meia
tela não ensina nada, e ainda faz a segunda aparição parecer enfeite —
[[Nota carrega só o que a pessoa não sabe]] aplicada ao próprio manual.

A barra fechada passa a dizer o que a faixa **não** diz: `Como usar · 7 passos, e 5
ressalvas`. É a informação que decide se vale abrir agora, e ela não existia em lugar
nenhum. O resumo desce pra dentro, como a linha de abertura do manual — quem abriu quer
o manual, e o manual começa dizendo pra que serve.

Regra geral que sai daí: **quando o topo da tela ganha uma frase, procure quem já dizia
a mesma frase mais abaixo.** A duplicata não nasce do bloco novo; ela nasce do antigo,
que continuou fazendo um trabalho que deixou de ser dele.

## O texto mora fora do componente

Uma constante por ferramenta, em `lib/how-to.tsx` — não dentro do JSX da tela. Revisar a
redação de três manuais não pode exigir desviar de layout. E o texto **importa a
constante** em vez de digitar o número (`IV_MAX`, não `32`): assim o manual não passa a
mentir depois de um ajuste na conta. Mesma lógica de
[[A tela não afirma mais precisão do que a fonte tem]].

## Onde não usar

Catálogo e ficha, que abrem cheios. E dentro do manual valem as regras de
[[Nota carrega só o que a pessoa não sabe]]: o passo diz o **porquê e a armadilha**, não
repete o rótulo do campo que ele descreve.

## Conexões
- Princípio: [[Tela que abre vazia tem que ensinar, tela que abre cheia não]] ·
  [[Nota carrega só o que a pessoa não sabe]] ·
  [[O que responde pergunta rara não ocupa a rolagem de todo mundo]]
- Irmã: [[Modal com conteúdo que cresce tem teto de altura e área que rola]] ·
  [[Padrões de componentes de dashboard]] ·
  [[Faixa de topo de ferramenta é chegada, não rótulo]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]]
