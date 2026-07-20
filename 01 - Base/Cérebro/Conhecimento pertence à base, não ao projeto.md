---
tags: [tipo/atomica, camada/principio, cerebro]
criado: 2026-07-20
---

# Conhecimento pertence à base, não ao projeto

> Uma nota de conhecimento nunca declara de qual projeto ela nasceu. O projeto
> aponta pra ela; ela não aponta de volta.

## A regra

Nota de princípio ou de padrão não tem campo `Origem: [[<projeto>]]` e não leva tag
`#projeto/*`. Quem linka é o projeto, na seção de aprendizados. O backlink do Obsidian
já mostra a origem sem que ela precise ser escrita — e mostra **todos** os projetos que
usaram aquilo, não só o primeiro.

## Por que

O cérebro tinha o vício inverso: quase toda nota de frontend terminava em
"Origem: [[Questor BI]]". Três efeitos ruins:

1. **O primeiro projeto virou dono.** Quem chegou depois (Navedesk, Cofre Digital)
   parecia estar pegando emprestado do Questor BI, quando na verdade os três bebem
   da mesma base. O grafo mostrava um sol com planetas em vez de um tronco com galhos.
2. **Conhecimento morre com o projeto.** Quando um projeto vai pro arquivo, tudo que
   pendurava nele vira nota órfã ou, pior, some do radar junto.
3. **O link vira ruído.** "Origem" é informação de arqueologia, não de uso. Na hora de
   construir algo novo eu quero saber *qual regra aplicar*, não *onde ela apareceu*.

## Na prática

Quando um aprendizado aparece no meio de um projeto, a pergunta certa não é "em que
projeto guardo isso?", é **"em que camada isso vive?"** — ver
[[Camadas do conhecimento - princípio, padrão, aplicação]]. Se for reutilizável, sobe pra
base e o projeto só linka. Se for específico daquele sistema (uma regra de negócio, um
número mágico do cliente), fica na nota do projeto mesmo e não vira atômica.

Exceção única: as notas de `Banco Questor/` levam `#banco/questor` porque são referência
de um **domínio externo** (o schema de um ERP), não de um projeto meu.

## Conexões
- Regra irmã: [[Camadas do conhecimento - princípio, padrão, aplicação]]
- Motivou: [[Manter o tooling enxuto e o conhecimento no cérebro]]
- Mapa: [[Base]]
