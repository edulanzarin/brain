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
- **Sem mismatch de hidratação**: no server e no 1º render client o `node` é null
  e o portal rende `null` (topo vazio); o ref anexa depois do commit → 2º render
  preenche. Server e 1º client batem. **Mas "sem mismatch" não é "renderizado no
  servidor"** — ver a seção abaixo, que é o preço.
- **Trocar de tela** desmonta o portal do módulo antigo e monta o do novo; o alvo
  (no Topbar) é estável e não pisca.

## O preço: o topo inteiro fica fora do HTML do servidor

Descoberto em set/2026, contra o que a seção acima sugeria. Portal existe depois
que o JavaScript roda, então **nada que o módulo injeta chega no HTML que o
servidor manda**: título da tela, abas, busca. Curl na página devolve a lista de
conversas pronta e o cabeçalho dela vazio.

As três consequências, em ordem de gravidade:

- **Sem JavaScript, o topo não existe nunca.** Não é degradação, é ausência: a
  pessoa vê a lista e nenhuma forma de filtrar ou trocar de aba.
- **O título aparece de repente.** O servidor pinta a página sem cabeçalho e ele
  entra na hidratação — o mesmo defeito de
  [[Conteúdo do servidor não pode nascer invisível esperando o cliente]], só que
  causado pelo mecanismo em vez de por uma classe.
- **Não dá para verificar por fora.** Conferir "esta pessoa vê a aba Equipe?" com
  uma requisição simples deixa de funcionar, porque a resposta não contém a aba.
  Some a checagem mais barata que existe sobre permissão na interface.

**Quando o portal ainda paga**: painel atrás de login onde o JavaScript é
pré-requisito e o topo carrega ação, não informação — um botão "novo", um filtro
de conforto.

**Quando não paga**: quando o topo carrega o QUE a tela está mostrando (título,
aba selecionada, termo buscado). Aí ele é conteúdo, e conteúdo sai do servidor.

O remédio é mais simples que o portal: **a barra de topo é do módulo, não do
layout**. Cada página renderiza o próprio cabeçalho como primeiro filho, e o
layout fica com o trilho e a área de conteúdo. Some o provider, some o portal,
some o contexto — e o cabeçalho passa a vir no HTML. O custo é o cabeçalho ficar
fora da área que rola, o que obriga a decidir quem rola: se o container do layout
rolar também, a tela com rolagem própria (uma inbox de dois painéis) ganha duas
barras de rolagem. Visto em [[navecrm]] (set/2026), que já nasceu com o cabeçalho no servidor.

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
- Irmã: [[Conteúdo do servidor não pode nascer invisível esperando o cliente]]
- Visto em: [[navetalks]] · [[navecrm]]
- Mapa: [[Design]]
