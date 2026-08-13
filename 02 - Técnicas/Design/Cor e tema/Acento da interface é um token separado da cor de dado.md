---
tags: [tipo/atomica, camada/padrao, design]
criado: 2026-08-13
---

# Acento da interface é um token separado da cor de dado

> Quando uma cor carrega significado de dado (azul = entrada), o acento de marca da interface tem de ser outro token — senão retematizar a UI contamina os gráficos.

## O problema

No dashboard, o mesmo `--ent` (azul) fazia dois papéis: a cor de **notas de entrada** nos KPIs, séries e barras (dado, com significado de domínio: azul entrada, verde saída) **e** o acento de toda a interface (nav ativa, botões, foco, checkbox, links). Enquanto a coincidência durou, ninguém percebeu que eram dois papéis.

Na hora de dar à interface uma cara própria (acento índigo), a conta veio: trocar `--ent` viraria os dados de entrada em índigo e quebraria a leitura azul=entrada/verde=saída; não trocar deixaria a UI presa à cor do dado. Uma classe (`text-ent`/`bg-ent`) aparecia ~270 vezes misturando os dois usos.

## A solução

Criar um token de **acento de interface** separado do token de **dado**, pelo papel e não pela cor — mesmo que hoje as cores coincidam:

```
--ent / --sai      cor de DADO (entrada/saída) — só gráficos, KPIs, barras, badges
--accent           cor de INTERFACE (nav, botão, foco, seleção, link)
```

A UI passa a usar `accent`; o dado mantém `ent`/`sai`. Retematizar a interface vira mexer num token só, sem tocar em nenhum gráfico. É o mesmo espírito de nomear pelo papel de [[Token semântico em vez de valor literal]]: "entrada" e "acento" são papéis distintos, logo tokens distintos.

## O que mais vale lembrar

Quando os papéis já nasceram fundidos, a triagem para separar é por **assinatura de uso**, não por busca cega:

- Vira acento (UI): nav/aba ativa, anel de foco, checkbox marcado, botão sólido (`bg + text-white`), link de ação (`hover:underline`), card/linha selecionada.
- Fica dado: série de gráfico, ícone/valor de KPI de entrada, barra de valor, badge de status categórico.
- **Na dúvida, é dado** — trocar um dado por engano corrompe a semântica; deixar um controle de UI na cor antiga é só um respingo.

## Conexões
- Princípio: [[Token semântico em vez de valor literal]]
- Irmã: [[Cor de marca precisa de variante acessível por tema]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Design]]
