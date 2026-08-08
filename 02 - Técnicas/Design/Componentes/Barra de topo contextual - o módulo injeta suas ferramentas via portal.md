---
tags: [tipo/atomica, camada/padrao, dev/frontend, design]
criado: 2026-08-08
---

# Barra de topo contextual - o módulo injeta suas ferramentas via portal

> A barra de topo não é fixa nem genérica: **cada módulo manda pra ela a própria
> barra de ferramentas** (busca, ações), e o topo muda conforme a tela. Assim
> some a busca duplicada (uma global no topo + uma em cada container) e o chrome
> global sobra só pro que é global.

## O problema

Quando o topo tem uma busca genérica ("buscar tudo") e **cada tela** ainda tem a
própria barra de busca dentro do container, o usuário vê duas caixas de busca na
mesma janela e a de cima quase nunca funciona de verdade. A ferramenta da tela
não tem um lugar canônico — está nos dois.

## A divisão

- **Rail (chrome global):** o que vale em qualquer tela — identidade (quem está
  logado / "ver como"), tema, navegação. Não muda com o conteúdo.
- **Topo (contextual):** título do módulo + a **barra de ferramentas daquela
  tela** (busca, "novo X", filtros). Muda a cada módulo.
- **Container:** só o conteúdo (a tabela, o fio, os cards). Sem barra própria.

## A técnica: slot por portal

Um provider guarda o nó de destino; o Topbar renderiza o **alvo**; cada módulo
renderiza um **portal** que joga seus controles lá dentro — mantendo estado e
handlers na árvore do módulo (o botão "novo" abre um modal que vive no módulo,
mesmo aparecendo no topo).

```tsx
const Ctx = createContext<{node,setNode} | null>(null);

export function TopbarSlotProvider({children}) {
  const [node,setNode] = useState<HTMLElement|null>(null);
  return <Ctx.Provider value={{node,setNode}}>{children}</Ctx.Provider>;
}
// dentro do Topbar, uma vez:
export function TopbarSlotTarget(p) {
  const ctx = useContext(Ctx);
  return <div ref={ctx?.setNode ?? null} className={p.className} />;
}
// dentro de cada módulo:
export function TopbarPortal({children}) {
  const ctx = useContext(Ctx);
  return ctx?.node ? createPortal(children, ctx.node) : null;
}
```

Detalhes que evitam dor:

- **Ref-callback como setState** (`ref={setNode}`) registra o nó quando ele monta
  e re-renderiza — o portal só dispara quando o alvo existe.
- **SSR/hidratação limpos**: no server e no 1º render client o `node` é null e o
  portal rende `null` (topo vazio); o ref anexa depois do commit → 2º render
  preenche. Server e 1º client batem, sem mismatch.
- **Trocar de tela** desmonta o portal do módulo antigo e monta o do novo; o alvo
  (no Topbar) é estável e não pisca.

## A busca do topo é um componente, não markup solto

A caixa de busca que vai no topo vira um componente próprio (`TopbarSearch`) com
**largura média fixa** e a sincronização com a URL (`?q=`) já embutida — o módulo
só declara `<TopbarSearch placeholder=… />` dentro do portal, sem repetir
debounce nem escrita de query a cada tela. O estado continua na URL
([[Filtro de lista mora na URL]]); o componente só padroniza a forma e o tamanho
(nada de barra full-width variando por tela — é o mesmo espírito de
[[A variante de um controle muda a intenção, não o tamanho]] aplicado à busca).

## Conexões
- Vizinha: [[Navegação de dois níveis - trilho de produto e sidebar de contexto]]
  — onde mora a navegação; esta nota é onde moram as *ferramentas da tela*.
- Parente: [[Filtro de lista mora na URL]] — o estado da busca continua na URL;
  o portal só muda **onde** o controle aparece, não onde o estado vive.
- Visto em: [[navetalks]]
- Mapa: [[Design]]
