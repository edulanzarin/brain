---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-08-12
---

# Janela arrastável no React 19 se faz à mão, não com react-rnd

> As libs de arraste/redimensão mais comuns (`react-rnd`, e a `react-draggable`
> por baixo) chamam `ReactDOM.findDOMNode`, **removido no React 19**. Num projeto
> React 19 elas quebram — a UI de janelas flutuantes se escreve à mão com pointer
> events, que é pouca coisa.

## Por que a lib não serve

`react-rnd` embrulha `react-draggable` + `re-resizable`; a `react-draggable`
localiza o nó arrastado por `findDOMNode(this)`. O React 19 tirou `findDOMNode` do
pacote — a chamada vira erro em runtime. Não é caso de "funciona em dev, quebra no
build": a API sumiu de vez. A lição de fundo: antes de adotar uma lib de UI de
baixo nível, conferir se ela pisa em algo que a versão do runtime ainda oferece.

## A forma de fazer à mão

Pointer events resolvem, e o segredo pra ficar fluido é **não passar cada frame
pelo estado do React**:

- **Durante o gesto, mexe no DOM** — no `pointermove` escreve `el.style.left/top`
  (ou `width/height`) direto no nó, via `ref`. Zero re-render por frame.
- **Comita no soltar** — só no `pointerup` chama o `setState` com a geometria
  final. É o único momento em que o React re-renderiza a janela.
- **Overlay imperativo** pro feedback do gesto (a prévia de encaixe): um `div`
  posicionado por `ref` no `pointermove` e escondido no fim — não vira estado,
  senão repinta as janelas no meio do arraste.
- **Efeito colateral de DOM vai no `useEffect`**, não no corpo do componente: o
  lint do React Compiler (`react-hooks/immutability`, Next 16) barra mutar um
  global (`document.body.style.userSelect`) direto no handler; um estado
  `arrastando` + efeito resolve, e só re-renderiza no início e no fim do gesto.

O encaixe (snap) é geometria pura: detecta a zona pela posição do ponteiro
relativa ao container (cantos antes das bordas) e aplica metade, quadrante ou
maximizar.

## Conexões
- Folha isolada (sem princípio ainda): a lição "checar o que a lib pisa antes de
  adotar" vira princípio quando aparecer um segundo caso.
- Visto em: [[Navetech Hub]]
- Mapa: [[Frontend]]
