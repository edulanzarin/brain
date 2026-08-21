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

1. **O resumo fica fora do fechado.** Ele é uma frase: *a pergunta que a ferramenta
   responde*. É o que o visitante de primeira viagem precisa; os passos são referência,
   e quem já usou não quer rolar por cima deles toda vez.
2. **`<details>`, não estado de React.** Abrir e fechar aqui é **chrome, não dado**: não
   vai pra URL (≠ [[Filtro de lista mora na URL]]), não precisa de efeito, não precisa de
   JS. Consequência prática no Next: o bloco continua **componente de servidor** e o
   manual nasce no HTML — inclusive pro buscador, que é quem traz gente nova. Teclado e
   leitor de tela vêm de graça.
3. **O tint é o da ferramenta**, o mesmo da navegação e da home, pra o manual ser lido
   como parte dela e não como aviso do site.

Detalhes que só aparecem montando: o `summary` inteiro é o alvo do clique (faixa, não
chevron de 16px); o marcador nativo sai com `list-none` **e**
`[&::-webkit-details-marker]:hidden`; e o resumo é `truncate` fechado, `whitespace-normal`
aberto, senão ele quebra a faixa em duas linhas em telas estreitas.

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
  [[Nota carrega só o que a pessoa não sabe]]
- Irmã: [[Modal com conteúdo que cresce tem teto de altura e área que rola]] ·
  [[Padrões de componentes de dashboard]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]]
