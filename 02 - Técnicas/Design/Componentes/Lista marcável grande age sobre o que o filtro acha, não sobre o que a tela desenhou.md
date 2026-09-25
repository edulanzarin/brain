---
tags: [tipo/atomica, camada/padrao, design]
criado: 2026-09-17
---

# Lista marcável grande age sobre o que o filtro acha, não sobre o que a tela desenhou

> Com 1.570 empresas numa lista de marcar, uma por uma não é ferramenta. O lote ("Marcar as 3") vale para tudo que a busca e a visão acharam, mesmo além das linhas desenhadas; a visão "Marcadas" revisa a seleção; Shift+clique pega intervalo; e Enter na busca não pode enviar o formulário.

## O problema

O seletor de empresas do grupo de permissão do Nexo desenhava 300 linhas e marcava uma a uma. Para montar "todas menos três" ou revisar as 92 que ficaram de fora, a única saída era rolar e clicar. E o campo de busca morava dentro do form: Enter nele salvava o grupo pela metade.

## A solução

- **Lote sobre o conjunto filtrado.** Sem busca, "Marcar todas / Desmarcar todas". Com busca ou visão, o rótulo diz o tamanho ("Marcar as 3") e a ação vale para as N filtradas, não só para as desenhadas. Desenhar é limite de performance; a ação não herda esse limite, e a nota de corte da lista diz isso ("Marcar e desmarcar valem para as 1.570").
- **Botão que não mudaria nada fica desabilitado**: tudo já marcado desliga "Marcar", nada marcado desliga "Desmarcar".
- **Visão Todas / Marcadas / Desmarcadas, com contagem.** Revisar a seleção é a tarefa mais comum depois de montar, e sem essa visão ela custa a lista inteira.
- **Shift+clique** marca ou desmarca do último clique até o atual, e o intervalo inteiro vai para o estado que o clicado passa a ter. Se a âncora sumiu da lista (busca mudou, visão escondeu), vale só o clicado: não se marca às cegas o que a pessoa não está vendo. `preventDefault` no `mousedown` com Shift, senão o navegador seleciona o texto das linhas.
- **Enter na busca** faz `preventDefault` (campo dentro de form submete com Enter) e, se sobrou uma só, marca ela: digitar o código e dar Enter vira o jeito rápido.
- **Contagens sobre a lista, não sobre o Set.** Código marcado que saiu do cadastro não aparece em visão nenhuma; contar o Set mostra um número que a tela não consegue explicar.

## Conexões
- Princípio: folha isolada por ora; candidata a promover "a ação não herda o limite do desenho" se aparecer em outra lista paginada ou virtualizada
- Irmã: [[Escolha única e múltipla não usam o mesmo controle]] · [[Grupo definido por exclusão guarda as de fora e se resolve na leitura]]
- Visto em: [[Navetech Hub]] (grupos de empresa em Admin e Configurações) · [[NaveX]] (a matriz de permissões do cargo: marcar o módulo com uma busca aberta vale só para as seções achadas, e é assim que se libera o Post Mortem de todos os módulos de uma vez)
- Mapa: [[Design]]
