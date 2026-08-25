---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-08-25
---

# Flutuante dentro de modal precisa vencer no z-index e no Escape

> Select, combobox e tooltip dentro de um modal quebram de duas formas ao mesmo tempo, e
> nenhuma das duas dá erro: a lista abre atrás do véu, e o Escape que devia fechar só a
> lista fecha o modal inteiro.

## O problema

Uma escada de camadas costuma nascer assim, e parece completa:

```
balão(30) < nav(40) < popover(50) < modal(100)
```

Ela carrega uma suposição que ninguém escreveu: **flutuante e modal nunca dividem a
tela**. No dia em que um formulário de modal ganha um combobox, a suposição cai — e como
os dois vão pro `body` por portal, eles viram irmãos no mesmo contexto de empilhamento. O
100 cobre o 50 e a lista simplesmente não aparece. Sem exceção, sem log, sem nada pra
procurar: o campo abre, o cursor pisca, e não vem opção nenhuma.

O segundo defeito é mais caro que o primeiro, porque destrói trabalho. Os dois escutam
`keydown` na fase de **captura** do `document`, e nessa fase o navegador chama os ouvintes
na ordem em que foram **registrados**. O modal abriu antes, então registrou antes, então
responde antes — e o `stopPropagation()` do flutuante nunca chega a rodar. Resultado:
Escape pra fechar a listinha fecha o modal e joga fora o formulário inteiro.

## A solução

**Desenho.** O flutuante fica acima do modal, e a regra é geral: flutuante é sempre filho
do controle que a pessoa está tocando AGORA, então ele pertence acima da camada onde esse
controle mora. Não existe caso em que o menu deva ficar sob o modal — se o modal abriu,
flutuante de trás dele já fechou.

```
balão(30) < nav(40) < modal(100) < popover(110)
```

**Teclado.** Não tente consertar por ordem de listener: ela depende de quem montou
primeiro, que é justamente o que você não controla. Quem está por baixo **pergunta** se há
camada acima antes de agir.

```ts
// popover.tsx
let abertos = 0;
export const temFlutuanteAberto = () => abertos > 0;
// abertos++ ao abrir, abertos-- na limpeza do efeito

// modal.tsx
if (e.key === "Escape") {
  if (temFlutuanteAberto()) return; // Escape fecha a camada DE CIMA
  onClose();
}
```

Contador e não booleano: flutuante empilha (combobox dentro de popover de filtro), e "tem
algum aberto" precisa continuar verdadeiro até o último fechar.

## O que mais vale lembrar

- **Duas modais empilhadas não são solução de navegação.** Abrir a segunda fecha a
  primeira: dois véus escurecem o fundo duas vezes, e com dois diálogos montados o Escape
  volta a depender de ordem de montagem.
- **O conserto é retroativo.** Toda peça que já usava flutuante dentro de modal estava
  quebrada em silêncio — no [[piwdex2]] havia um tooltip invisível num modal de perfil,
  que ninguém tinha reportado porque tooltip que não aparece parece "não tem tooltip".
- **A escada mora num lugar só.** Se os números estão espalhados por classe utilitária em
  dez arquivos, a próxima camada nova repete o problema. Deixe o comentário da escada
  onde ele é lido: junto do componente que a define.

## Conexões
- Princípio: [[Todo estado da tela tem visual]]
- Irmã: [[Modal com conteúdo que cresce tem teto de altura e área que rola]] ·
  [[Portal condicional dispensa o flag de montagem]]
- Visto em: [[piwdex2]]
- Mapa: [[Frontend]]
