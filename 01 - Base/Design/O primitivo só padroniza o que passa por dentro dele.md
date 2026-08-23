---
tags: [tipo/atomica, camada/principio, design, armadilha]
criado: 2026-08-24
---

# O primitivo só padroniza o que passa por dentro dele

> Ter uma biblioteca de primitivas não garante consistência. Ela garante
> consistência **nas telas que a chamam**. Uma cópia local — cinco linhas de
> `<span>` com largura em porcentagem — não aparece em nenhum inventário, não
> quebra nenhum teste, e produz exatamente o sintoma que a biblioteca existia
> para evitar.

## Como a cópia nasce

Nunca por decisão. Nasce assim: a primitiva parece grande demais para o caso,
ou falta uma prop, ou é mais rápido escrever a barra ali do que abrir o arquivo
dela. Cada cópia é local e defensável; o conjunto é uma tela que mede as coisas
de um jeito e o resto do produto de outro.

No [[piwdex2]] eu escrevi um medidor próprio no painel do robô. Quando o Eduardo
apontou, corrigi o wrapper — e deixei uma segunda cópia dentro do modal de
pokémon, que é justamente onde há mais barras na tela. Ele apontou de novo. **A
correção de uma cópia não encontra as outras**, porque o que as une é a ausência
de um vínculo.

O mesmo aconteceu com altura: um selo desenhado à mão em `h-6` ao lado de badges
que a biblioteca cria em `h-7`. Oito pixels, visíveis, e insistentes.

## O que fazer

- **Procure por FORMA, não por nome.** `grep` pelo padrão visual (`overflow-hidden`
  numa barra, `h-6` num selo, `width: ${x}%`) encontra o que buscar pelo nome do
  componente nunca acha.
- **Se falta uma prop, abra a prop.** Foi a decisão que a cópia estava evitando, e
  é a única que conserta as outras telas junto.
- **Escadas de tamanho são contrato.** Se os selos do sistema vivem em `h-6`/`h-7`,
  qualquer coisa que apareça ao lado deles herda a escada — ou desalinha por
  construção.
- **Desconfie de "só desta vez".** A cópia não custa quando é escrita; custa
  quando alguém corrige a original e o produto fica meio corrigido.

## Conexões
- Irmã: [[A classe do chamador só vence a do primitivo com tailwind-merge]] ·
  [[Primitiva de botão fecha o tamanho e abre só a variante]] ·
  [[A variante de um controle muda a intenção, não o tamanho]]
- Visto em: [[piwdex2]]
- Mapa: [[Design]] · [[Base]]
