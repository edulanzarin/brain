---
tags: [tipo/atomica, camada/padrao, dev/backend]
criado: 2026-09-17
---

# Grupo definido por exclusão guarda as de fora e se resolve na leitura

> "Todas as empresas menos o próprio escritório" montado como lista marcada é a foto do cadastro no dia da montagem: toda empresa cadastrada depois fica de fora até alguém editar o grupo. O grupo guarda um **modo** (`lista` ou `exceto`) e as marcadas; no `exceto`, as marcadas são as de fora e os membros saem de universo − marcadas na hora de ler.

## O problema

No Nexo, o grupo de permissão "Todas - Navecon, Four, Finave" eram 1.478 empresas marcadas uma a uma. Quem tinha o cargo não enxergava a empresa que o escritório acabara de cadastrar, e o admin tinha que caçar as novas para marcar. O grupo nasceu de um seed que já avisava "é um snapshot, rode de novo para reconciliar": o defeito estava documentado como rotina.

## A solução

- **Mesma tabela de itens, significado pelo modo.** `modo text check (modo in ('lista','exceto'))`, default `lista`, então todo grupo existente continua igual sem migrar dado.
- **Um resolvedor só** diz quem está no grupo (`resolverGrupos`), e todo consumidor passa por ele: escopo da sessão, filtro por grupo, contagem da tela. Se sobrar um `select ... from grupo_item` solto, ele volta a tratar as marcadas como o grupo e inverte o `exceto` em silêncio.
- **O universo vem da fonte**, o cadastro de empresas do ERP, só quando algum grupo é `exceto`, com cache curto em memória (5 min): a sessão resolve a cada requisição e uma tela dispara várias.
- **Trocar de modo inverte as marcações** (`marcadas = universo − marcadas`), em vez de reinterpretá-las. O grupo sai da troca com as mesmas empresas: as 1.478 marcadas viram 92 de fora. Reinterpretar deixaria de fora justamente as 1.478.
- **Dica no lugar do erro comum**: lista com mais da metade marcada sugere trocar para "Todas, exceto".

## O que mais vale lembrar

**Script que regrava itens precisa saber o modo.** O seed antigo gravava a lista de DENTRO; rodado sobre um grupo já trocado para `exceto`, gravaria a de dentro como a de fora e inverteria o grupo inteiro. O seed passou a gravar as de fora e cravar o modo no insert e no conflito. Todo escritor da tabela de itens é consumidor do modo, não só os leitores.

## Conexões
- Princípio: [[Estado mutável se lê da fonte no uso, não de cópia guardada]]
- Irmã: [[Escopo de dado se clampa no servidor, num funil só]]
- Visto em: [[Navetech Hub]] (grupos de empresa de permissão e de negócio)
- Mapa: [[Backend]]
