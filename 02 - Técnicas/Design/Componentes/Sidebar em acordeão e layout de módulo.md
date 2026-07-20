---
tags: [tipo/atomica, camada/padrao, design, dev/frontend]
criado: 2026-07-20
---

# Sidebar em acordeão e layout de módulo

> A estrutura fixa da tela: navegação que escala pra muitos módulos e um layout que
> segura o que é compartilhado entre as seções.

## Sidebar em acordeão

Módulos que expandem e colapsam, com o módulo da rota atual aberto por padrão. Escala
pra muitos módulos sem virar paredão de links — que é o destino de toda sidebar plana
depois do quinto item.

A configuração das seções mora num array só:

```ts
{ id, rotulo, path, metrica, descricao }
```

Esse array dirige **sidebar, header e regras de exibição** ao mesmo tempo. Módulo novo
é uma entrada no array, não três lugares pra editar e um pra esquecer.

## Layout de módulo

O layout segura o que é compartilhado (header e barra de filtros); cada seção é uma
rota própria que busca **só os seus dados**. Isso mantém o filtro visível e estável
enquanto o conteúdo troca, e evita que a página inteira recarregue pra mudar de aba.

Consequência importante: o estado que a barra de filtros controla pertence à seção, não
à página — ver [[Estado de tela pertence à seção, não à página]].

## Quando o acordeão graduou para launcher

O acordeão sobre todos os módulos serve enquanto são poucos e todo mundo vê todos.
Quando os módulos viram unidades com **permissão própria** (um usuário acessa Contábil
mas não Fiscal), a navegação muda de forma: uma tela **launcher** na raiz escolhe o
módulo — mostrando só os que a sessão libera — e, dentro dele, a sidebar é **escopada**
(só as seções daquele módulo, com "trocar módulo" pra voltar). O launcher vira, de
brinde, a primeira porta de permissão.

O que não muda é a ideia de fundo: um **catálogo único** dirige launcher, sidebar e
gate ao mesmo tempo; cada módulo tem seu `layout` com a sidebar dele; cada seção busca
só os seus dados. Foi a troca feita no [[Questor Hub]] ao sair de um módulo para vários
com login à vista. O gate que o launcher inaugura em
[[Cravar o seam de permissão antes do login]].

## Conexões
- Princípio: [[Container tem largura máxima e respiro constante]]
- Índice: [[Padrões de componentes de dashboard]]
- Irmãs: [[Controles de filtro do dashboard]] · [[Blocos de dado - card, KPI e gráfico]]
- Mapa: [[Design]]
