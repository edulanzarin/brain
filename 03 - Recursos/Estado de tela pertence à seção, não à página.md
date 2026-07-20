---
tags: [tipo/atomica, dev/frontend, conceito, projeto/questor-bi]
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

A saída é guardar o estado **por seção do menu lateral**, fora da árvore React:

```ts
const estados = new Map<string, unknown>();  // chave: `${secao} ${campo}`
```

com um `useState` que restaura da chave e escreve nela a cada mudança. A seção
sai do `pathname`, então a tela só declara o nome do campo:

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

Duas armadilhas ao migrar:

- **Efeito que zera a paginação** (`useEffect(() => setPagina(1), [filtro])`)
  roda também na remontagem e joga fora a página guardada. Comparar com o
  recorte anterior num `ref` — remontar não é mudar.
- **Campos de mesmo nome em abas diferentes** colidem, porque a chave é da
  seção. Onde o sentido difere (a "conta" que se importa × a que se cadastra),
  prefixar o nome.

Vale também para o estado caro: um extrato bancário já lido e casado sobrevive
à ida à aba de regras, sem pedir o arquivo de novo.

## Conexões
- Apareceu em: [[Questor BI]], nas seções do módulo Contábil
- Ver também: [[Cache do React Query não é lugar de estado de interface]]
- Faz parte de: [[Desenvolvimento]]
