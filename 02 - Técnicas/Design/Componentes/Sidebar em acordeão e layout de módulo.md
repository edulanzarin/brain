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

## Conexões
- Princípio: [[Container tem largura máxima e respiro constante]]
- Índice: [[Padrões de componentes de dashboard]]
- Irmãs: [[Controles de filtro do dashboard]] · [[Blocos de dado - card, KPI e gráfico]]
- Mapa: [[Design]]
