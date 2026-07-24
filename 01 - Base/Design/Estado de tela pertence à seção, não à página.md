---
tags: [tipo/atomica, camada/principio, dev/frontend]
criado: 2026-07-20
---

# Estado de tela pertence à seção, não à página

> Busca, filtro e página são estado local do componente — e componente
> desmonta ao navegar. Se as telas são abas do mesmo assunto, o dono do estado
> é a seção, não a página.

## O quê

Numa tela com abas (`Conferência` | `Configuração`), cada aba é uma rota. Ir e
voltar desmonta e remonta a página, então todo `useState` volta ao inicial: a
busca digitada, o filtro escolhido, a página da tabela.

O usuário não vê duas telas — vê um assunto só. E o fluxo real vai e volta:
ver a pendência numa aba, ajustar a regra na outra, voltar. Refazer o recorte
a cada volta é trabalho jogado fora.

A saída é guardar o estado fora da árvore React, num `Map` de módulo, com uma
chave de três partes:

```ts
const estados = new Map<string, unknown>();  // chave: `${secao} ${pagina} ${campo}`
```

O ponto fino está em **separar identidade de tempo de vida**:

- **página + campo** é a *identidade* — é o que evita a `busca` de uma aba
  aparecer na outra;
- **seção** é o *tempo de vida* — está na frente da chave só para limpar a
  seção inteira de uma vez ao sair dela.

Confundir os dois é o erro fácil (foi o meu): chavear por `seção + campo`
funciona até duas abas da mesma seção usarem o nome `busca`. Aí o usuário
filtra numa aba e a outra aparece filtrada sozinha — pior que não persistir,
porque esconde dados com um filtro que ele não pôs ali.

Seção e página saem as duas do `pathname`, então a tela só declara o campo:

```ts
const [busca, setBusca] = useEstadoSecao("busca", "");
```

Quem **descarta** é o shell da seção, no cleanup do efeito — é ele que sabe
qual seção está ativa, e o cleanup cobre tanto trocar de seção quanto sair do
módulo.

## Por que importa

Guardar é metade do problema; a outra metade é **quando soltar**. Persistir
para sempre é pior que não persistir: voltar dias depois numa tela
pré-filtrada esconde dados sem avisar, e o usuário não lembra que filtrou. A
regra que se sustenta é "vale enquanto estou no assunto".

Armadilha ao migrar: o **efeito que zera a paginação**
(`useEffect(() => setPagina(1), [filtro])`) roda também na remontagem e joga
fora justamente a página guardada. Comparar com o recorte anterior num `ref` —
remontar não é mudar.

E uma lição de projeto de API: a primeira versão pedia prefixo manual no nome
do campo (`"regras.conta"`) para escapar da colisão. Convenção que depende de
lembrar não é solução — some na primeira tela nova. Melhor a chave carregar a
página sozinha e o nome do campo ficar burro.

Vale também para o estado caro: um extrato bancário já lido e casado sobrevive
à ida à aba de regras, sem pedir o arquivo de novo.

## Compartilhado entre shell e página

Evolução (jul/2026): o store ganhou **ouvintes** e o hook lê via
`useSyncExternalStore` — duas instâncias do MESMO campo ficam em sincronia.
Isso destrava um layout novo: os controles de uma aba (conta, arquivo) podem
ser renderizados **pelo shell, na linha da barra de filtros**, e a página os
enxerga na hora, porque ambos leem e gravam o mesmo campo da seção. Sem
prop-drilling, sem portal, sem contexto novo: o mecanismo que já dava tempo de
vida ao estado passou a dar também o compartilhamento.

## Conexões
- Ver também: [[Cache do React Query não é lugar de estado de interface]]
- Visto em: [[Navetech Hub]], nas seções do módulo Contábil
- Mapa: [[Base]] · [[Design]]
