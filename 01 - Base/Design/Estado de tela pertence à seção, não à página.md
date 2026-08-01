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

Quem **descarta** é o shell do módulo, no cleanup do efeito de unmount — trocar
de seção mantém, só "Trocar módulo" solta. (O tempo de vida subiu da seção para
o módulo; ver o fim da nota.)

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

## Tempo de vida subiu da seção para o módulo (jul/2026)

Refinamento do "quando soltar": o dono do tempo de vida não é a **seção**, é a
**fronteira de layout que sobrevive à navegação que o estado precisa
atravessar** — no app, o **shell do módulo**. Ele persiste ao trocar de seção e
só desmonta em "Trocar módulo", então é o unmount dele que descarta, não a troca
de seção.

O que forçou a correção: ir da Conferência ao Balancete (duas seções do mesmo
módulo) e voltar jogava fora o extrato/prévia já importado — dado que **não dá
pra reconstruir da URL** —, obrigando a reimportar. O fluxo real de trabalho
atravessa seções irmãs, não só as abas de uma.

Estender o tempo de vida é **seguro** graças à própria separação identidade ×
tempo de vida: a chave já carrega a seção, então cada seção guarda o seu recorte
e uma não vê o da outra mesmo todas vivas ao mesmo tempo. Só o *quando soltar*
mudou — a identidade ficou igual.

A memória de filtro (o `ap` e o recorte na URL, guardados por seção) encurtou no
mesmo movimento — de "sessão inteira" para "módulo" —, pra reentrar num módulo
começar limpo em vez de restaurar recorte de outra visita. O módulo virou a
unidade que persiste e zera junto, alinhando Contábil/Fiscal/Folha ao RH, que já
descartava por módulo (`use-estado-modulo`). O medo antigo ("voltar dias depois
numa tela pré-filtrada") continua válido — só que o ponto certo de soltar é
**sair do módulo**, não sair da seção: dentro do módulo você está no trabalho, e
aí lembrar é recurso, não armadilha.

## Conexões
- Ver também: [[Cache do React Query não é lugar de estado de interface]]
- Visto em: [[Navetech Hub]], nas seções do módulo Contábil
- Mapa: [[Base]] · [[Design]]
