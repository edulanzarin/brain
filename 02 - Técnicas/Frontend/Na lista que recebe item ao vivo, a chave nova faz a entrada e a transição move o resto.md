---
tags: [tipo/atomica, camada/padrao, dev/frontend]
criado: 2026-09-24
---

# Na lista que recebe item ao vivo, a chave nova faz a entrada e a transição move o resto

> Pilha das últimas vendas: quando uma venda chega, ela cai na frente e as outras
> escorregam para trás. Parece pedir biblioteca de animação de lista. Não pede: o
> React já separa quem chegou de quem só mudou de lugar, pela chave.

## Como funciona

Com `key` estável (o id da venda), a reconciliação faz duas coisas diferentes:

- o item com **chave nova** é montado do zero, então uma animação CSS de entrada
  (`animation: chega 700ms backwards`) roda nele, e só nele;
- os itens com **chave já conhecida** mantêm o nó do DOM e só recebem estilo novo
  (a posição na pilha). Uma `transition` em `transform` e `opacity` anima a troca.

```tsx
{vendas.slice(0, 3).map((v, i) => (
  <Ingresso
    key={v.id}
    chegou={i === 0}
    className="absolute inset-x-0 top-0 transition-[transform,opacity] duration-500"
    style={{ transform: `translateY(${i * 12}px) scale(${1 - i * 0.05})`, opacity: 1 - i * 0.3, zIndex: 10 - i }}
  />
))}
```

Funciona igual quando a lista vem do servidor: `router.refresh()` depois de um evento
ao vivo re-renderiza o componente de servidor, e a reconciliação do payload respeita as
chaves do mesmo jeito.

## Dois cuidados

- **A entrada usa propriedades individuais** (`translate`, `rotate`, `scale`) e não
  `transform`, porque `transform` já está ocupado pela posição na pilha. As duas se
  somam em vez de uma apagar a outra. Mesma raiz de
  [[Entrada animada com preenchimento both prende o transform e anula o hover]].
- **Quem sai não anima**: o item empurrado para fora das três posições desmonta na
  hora. Na pilha isso passa, porque ele já estava quase transparente atrás. Se a
  saída importar, aí sim é caso de biblioteca ou de `View Transitions`.

## Conexões
- Princípio: [[Dado que chega preenche espaço reservado, não empurra a tela]]
- Depende de: [[Estado vivo se empurra, não se pergunta]]
- Irmã: [[Entrada animada com preenchimento both prende o transform e anula o hover]]
- Visto em: [[telebot]]
- Mapa: [[Frontend]]
