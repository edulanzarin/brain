---
tags: [tipo/atomica, camada/padrao, design, dev/frontend]
criado: 2026-07-20
---

# Consulta pesada executa por botão, não por mudança de filtro

> Mudar um filtro não dispara consulta: o usuário edita um rascunho e um botão
> comita. Ninguém escolhe todos os filtros ao mesmo tempo — o auto-fetch roda
> sempre cedo demais, com o recorte pela metade.

## O problema

Com auto-fetch, escolher a empresa já dispara a consulta antes de o período ser
escolhido — uma query pesada com recorte errado, que trava a tela e ainda será
jogada fora. O usuário perde o controle do custo.

## O mecanismo: rascunho e aplicado

Dois estados com papéis distintos:

- **Rascunho** (local): o que o usuário está editando. Mudar não consulta nada.
- **Aplicado** (na URL, com um marcador `ap=1`): o que dirige as queries — e
  continua compartilhável/sobrevivendo ao reload, ver
  [[Estado compartilhável mora na URL]].

O botão comita rascunho→aplicado. Antes do primeiro comite, a tela mostra um
estado "ajuste e execute" (nada de dados); um hook único (`useRascunhoFiltros`)
serve todas as barras de filtro — mecanismo um, telas muitas.

## O verbo segue a ação real da tela

Botão genérico em tudo é regra cega. O catálogo de telas declara a execução:

- **"Executar"** onde a consulta computa algo (conferência, painéis);
- **"Carregar"** onde só traz um cadastro pronto;
- **gatilho próprio** onde o insumo é da tela — na conciliação bancária o
  filtro de empresa aplica na hora (é contexto, não consulta), e o Executar
  processa o **arquivo escolhido**. O refinamento importante: **escolher o
  arquivo não processa** — só guarda e mostra o nome; o processamento espera o
  botão, como qualquer consulta. Escolher é escolher, executar é executar.

## Feedback no ponto do clique

O clique precisa de resposta onde aconteceu: o botão vira spinner
("Executando…") e trava enquanto a consulta roda — filtrando as queries de
suporte da própria barra (lista de empresas), que não podem fazê-lo girar
sozinho. Skeleton cobre o primeiro load da área; ver
[[Esqueleto de carregamento imita a forma do conteúdo]] e
[[Todo estado da tela tem visual]].

## Conexões
- Princípio: [[Todo estado da tela tem visual]] · [[Estado compartilhável mora na URL]]
- Irmã: [[Controles de filtro do dashboard]] · [[Filtro de lista mora na URL]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Design]]
