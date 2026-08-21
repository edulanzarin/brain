---
tags: [tipo/atomica, camada/padrao, design, dev/frontend, armadilha]
criado: 2026-08-20
---

# Conteúdo do servidor não pode nascer invisível esperando o cliente

> O jeito natural de fazer uma imagem aparecer suave é `opacity-0` até o `onLoad`. Num app
> renderizado no servidor isso significa que a página chega pronta, correta e **em branco**
> — porque quem acende a luz é um estado de React que ainda não existe.

## O problema

O padrão é inocente:

```tsx
const [ok, setOk] = useState(false);
<img onLoad={() => setOk(true)} className={ok ? "opacity-100" : "opacity-0"} />
```

Em SPA passa despercebido: nada existe antes do JS mesmo. Em SSR/SSG, não. O HTML sai do
servidor com a `src` certa, o navegador já baixa e já pinta a imagem — e o CSS a esconde,
esperando um `useState` que só vai virar depois de baixar o bundle, hidratar e disparar o
evento.

O resultado é uma grade de 60 placeholders numa página cujo conteúdo **já estava lá**.
Pior: em rede ruim, JS bloqueado ou crawler, o estado nunca vira, e o conteúdo nunca
aparece. E não dá pra flagrar em dev, onde a hidratação leva 40ms.

## A solução

Inverter a pilha: o conteúdo nasce **visível**, o marcador fica **atrás**.

```tsx
<span className="relative grid place-items-center">
  {estado !== "ok" ? <Marcador className="absolute" /> : null}
  {src && estado !== "fail" ? (
    <img src={src} onLoad={() => setOk()} onError={() => setFail()}
         className="relative h-full w-full object-contain" />
  ) : null}
</span>
```

Enquanto a imagem não pinta, ela é transparente e o marcador aparece por baixo. Quando
pinta, ela cobre. Sem JS, o caminho feliz continua funcionando — o JS passa a servir só
pro caso de FALHA, que é o único que o HTML sozinho não sabe tratar.

No mesmo componente, a irmã dessa armadilha: **`onLoad` pode nunca chegar**, porque a
imagem costuma terminar de carregar antes do React ligar o listener. O mount tem que
conferir `img.complete`, que é a verdade do DOM, em vez de confiar só no evento.

## O que mais vale lembrar

A regra generaliza pra além de imagem: **animação de entrada, `reveal` no scroll, conteúdo
que só aparece depois de um `useEffect`**. Se a técnica de transição parte de "invisível",
ela apagou a página renderizada no servidor.

O teste é barato e definitivo: `curl` na URL e procurar o conteúdo no HTML. Se a tag está
lá com `opacity-0`, o SSR virou enfeite.

## Visto em

No piwdex2 a tabela da Pokédex renderizava 17 linhas de pokebola cinza no lugar dos
sprites; o HTML do servidor trazia `<img src="/game-sprites/58.webp" ... class="opacity-0">`
com a imagem já disponível e local. Invertida a pilha, o sprite passou a aparecer no
primeiro paint, sem JS nenhum.

## Conexões
- Princípio: [[Dado que chega preenche espaço reservado, não empurra a tela]]
- Irmã: [[Esqueleto de carregamento imita a forma do conteúdo]] · [[Slot com placeholder esmaecido segura o lugar do dado vivo]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]] · [[Frontend]]
