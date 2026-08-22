---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-08-21
---

# Slot de anúncio no App Router precisa de casca estável e filho keyado

> Um `<ins class="adsbygoogle">` só recebe anúncio uma vez: o script do Google
> reivindica o nó e escreve `data-adsbygoogle-status="done"`. Num app que
> **reconcilia** em vez de remontar — que é o App Router entre rotas de mesma
> forma — o nó sobrevive à navegação com o anúncio velho dentro, e nenhum `push`
> novo funciona. A forma que resolve é sempre a mesma: **casca estável que reserva
> o espaço, filho keyado que carrega o `<ins>` e o efeito.**

## As quatro armadilhas, e por que cada uma some sozinha

```tsx
export function Anuncio({ lugar, minH }: Props) {
  const pathname = usePathname();
  return (
    <div style={{ minHeight: minH, contain: "layout" }}>   {/* casca: nunca remonta */}
      <Unidade key={`${lugar}:${pathname}`} lugar={lugar} />  {/* filho: morre na troca */}
    </div>
  );
}

function Unidade({ lugar }: { lugar: string }) {
  const empurrado = useRef(false);                    // guarda que sobrevive ao StrictMode
  useEffect(() => {
    if (empurrado.current) return;
    empurrado.current = true;
    (window.adsbygoogle = window.adsbygoogle || []).push({});
  }, []);
  return <ins className="adsbygoogle" style={{ display: "block", width: "100%" }} … />;
}
```

1. **StrictMode empurra duas vezes.** O efeito roda setup → cleanup → setup no
   MESMO nó, e o segundo `push` derruba a unidade com *"All ins elements in the
   DOM with class=adsbygoogle already have ads in them"*. A guarda tem que ser
   `useRef` — guardar por atributo do DOM falha, porque enquanto o script não
   baixou os dois pushes só entram na fila e o erro estoura depois.
2. **`/dex/1` → `/dex/2` não remonta nada.** Mesma árvore, mesma posição: o React
   reutiliza o `<ins>` já reivindicado e o efeito com `[]` nem roda. O sintoma é
   silencioso — o mesmo anúncio para sempre, uma impressão onde deveriam ser
   várias. Pôr `pathname` na lista de dependências não resolve: o elemento
   continua reivindicado, e o push dá erro. **Quem mata o nó é a `key`.**
3. **A `key` leva a rota e SÓ a rota.** Keyar por parâmetro de busca faz o
   anúncio recarregar a cada clique de filtro — pedido novo por interação, que é
   o padrão que o Google trata como tráfego inválido.
4. **Numa lista filtrada, o marcador de anúncio carrega ORDINAL, não posição.**
   Com `key={índice}`, mexer no filtro remonta o slot e pede outro anúncio; com
   `key={n-ésimo anúncio}`, ele atravessa a filtragem vivo.

## O gate é a variável de ambiente, e ela é de BUILD

`NEXT_PUBLIC_*` é inlinado no `next build`. Isso dá o melhor interruptor
possível: sem a variável, a checagem vira código morto e o navegador não recebe
uma linha de anúncio — nem script, nem requisição, nem impressão inválida em
`localhost` (que o Google conta contra você).

O corolário morde no Docker: `environment:` no compose é RUNTIME e **não chega no
bundle do cliente**. Tem que entrar como `build.args` → `ARG`/`ENV` antes do
`npm run build`. Ver [[Variável de ambiente do cliente é decidida no build, não no deploy]]
se ela já existir; senão, é este parágrafo.

## O que o site precisa ter, além do código

`ads.txt` na raiz (`public/ads.txt`, com `pub-…` e não `ca-pub-…`), `robots` que
não bloqueie nem `/ads.txt` nem o `Mediapartners-Google` (é ele que lê a página
pra escolher anúncio contextual), sitemap — página dinâmica que ninguém enxerga
vira "site sem conteúdo" na revisão — e política de privacidade dizendo que
terceiros usam cookies. A falta desses quatro reprova mais gente que erro de
código.

## Conexões
- Princípio: [[Verificar no build de produção, não só em dev]]
- Irmã: [[Anúncio em feed não pode vestir a roupa do conteúdo]] ·
  [[Token de cor que não existe vira cor herdada, sem erro]]
- Visto em: [[piwdex2]]
- Mapa: [[Frontend]]
