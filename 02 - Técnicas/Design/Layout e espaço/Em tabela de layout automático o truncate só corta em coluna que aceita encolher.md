---
tags: [tipo/atomica, camada/padrao, design, dev/frontend, armadilha]
criado: 2026-09-24
---

# Em tabela de layout automático o truncate só corta em coluna que aceita encolher

> Numa `<table>` com `table-layout: auto`, a largura da coluna é preferência, e
> o texto longo alarga a coluna até caber. O `truncate` do conteúdo nunca corta.
> Para cortar, a célula precisa aceitar encolher (`max-width: 0`), e isso só
> funciona bem em coluna de texto com largura em porcentagem.

## O problema

A tabela de funcionários do NaveX saiu com o CPF quebrado em duas linhas e a
última coluna fora da tela, mesmo com todo nome dentro de `truncate`.

## O que não funcionou

- **`max-width: 0` em toda célula que trunca** (`td:has(> .truncate)`): o
  navegador reparte a sobra pela largura máxima de cada coluna, e a coluna de
  critério da auditoria, que cabia inteira, passou a cortar no meio da frase.
- **`max-width: 0` em toda coluna com largura declarada**: a coluna de pixels
  (CPF, 140px) foi espremida abaixo do que declarou, e o texto invadiu a vizinha.

## A regra que ficou

1. Célula em uma linha só (`white-space: nowrap`): é a linha enxuta, e sem isso a
   data e o CPF quebram no separador.
2. Coluna de **texto com largura em porcentagem** encolhe e trunca nela
   (`max-width: 0` só nessa célula).
3. Coluna com largura em **pixels** mantém o tamanho; coluna **numérica** nunca
   trunca.
4. Coluna sem largura tem o tamanho do conteúdo. Se a soma passar do contêiner, a
   tabela rola na horizontal, com a pista nas bordas.

Na primitiva de tabela isso é uma linha: a classe entra quando a largura da
coluna termina em `%` e a coluna não é alinhada à direita.

## Conexões
- Princípio: [[Propriedade escolhida pelo visual redefine a estrutura por baixo]]
- Irmã: [[Rolagem horizontal que não se anuncia esconde a coluna que decide]]
- Visto em: [[NaveX]]
- Mapa: [[Design]]
