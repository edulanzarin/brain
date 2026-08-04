---
tags: [tipo/atomica, camada/padrao, design, dev/frontend]
criado: 2026-08-04
---

# Navegação de dois níveis - trilho de produto e sidebar de contexto

> Quando um produto vira suíte (vários produtos sob a mesma casca), a navegação ganha
> dois níveis: um trilho de produto e uma sidebar contextual do produto ativo.

## O problema

Uma sidebar plana (ou em acordeão) segura bem enquanto os módulos são de UM produto.
Quando o app cresce pra uma **suíte** — Atendimento, CRM, Tarefas, Automações no mesmo
lugar —, empilhar tudo numa lista só vira paredão e mistura contextos sem relação. Mas
jogar o usuário numa tela launcher a cada troca também irrita: ele quer pular de
produto e continuar navegando na mesma casca.

## A solução

Dois níveis persistentes, lado a lado:

- **Nível 1 — trilho de produto** (só ícones, à esquerda): troca a ÁREA ativa
  (Atendimento, CRM…). Produto novo é só mais um ícone. É o "launcher" que graduou de
  tela para trilho fixo — ver [[Sidebar em acordeão e layout de módulo]].
- **Nível 2 — sidebar contextual** (com rótulos, recolhível): mostra os MÓDULOS da
  área ativa e troca junto com o nível 1.

Um **catálogo único em dado** dirige os dois — a definição é `áreas → módulos`, não
código espalhado:

```ts
AREAS = [{ id, label, icon, home, modules: [{ id, label, href, icon, countKey }] }]
```

Área ou módulo novo é uma entrada no array, não três lugares pra editar e um pra
esquecer. Configurações pode ser uma **área global** (rodapé do trilho) com seções
deep-linkáveis (`?s=`), e a tela lê a seção da URL — a sidebar contextual é quem
navega.

## O que mais vale lembrar

- O trilho é a primeira porta natural de **permissão**: mostrar só as áreas que a
  sessão libera (mesma ideia do launcher em [[Cravar o seam de permissão antes do login]]).
- O breadcrumb sai de graça: área (na sidebar) > módulo (na topbar).
- Estado da seção mora na URL, não em estado de componente — sobrevive a reload e é
  compartilhável ([[Estado de tela pertence à seção, não à página]]).

## Conexões
- Princípio: [[A definição em dado dirige o comportamento, não um caso no código]]
- Irmã: [[Sidebar em acordeão e layout de módulo]]
- Índice: [[Padrões de componentes de dashboard]]
- Visto em: [[Navehub]]
- Mapa: [[Design]]
