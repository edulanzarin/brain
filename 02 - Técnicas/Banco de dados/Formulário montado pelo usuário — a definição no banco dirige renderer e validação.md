---
tags: [tipo/atomica, camada/padrao, dev/backend, sql]
criado: 2026-07-27
---

# Formulário montado pelo usuário — a definição no banco dirige renderer e validação

> Um builder tipo Google Forms: a estrutura do formulário é **dado** (tabela de
> campos tipados + `config` jsonb por tipo), e uma única peça de código renderiza,
> pré-visualiza e valida. O usuário cria e muda formulário sem migration nem deploy.

## O problema

Formulário fixo no código (campos e opções como constantes) não deixa quem entende
do negócio — o RH, aqui — criar ou mudar nada; cada campo novo é código + migration.
E logo aparecem N formulários diferentes.

## A solução

Duas tabelas: `formulario` (nome, status) e `formulario_campo` (`ordem`, `tipo`,
`rotulo`, `obrigatorio`, `config jsonb`). O `tipo` é um enum fechado
(`texto_curto | texto_longo | selecao_unica | selecao_multipla | nota | pontuacao`)
e o que varia por tipo mora no `config` — `{opcoes}`, `{escala}`, `{min,max}` — então
o formulário evolui sem migration. Um campo pode marcar `config.papel: 'decisao'`
para virar destaque nos painéis.

Três peças, uma fonte:
- **um renderer** (`CamposFormulario`) desenha cada tipo a partir da definição;
  serve o preview do builder E o formulário público, controlado por `valores`.
- **uma validação pura** (`validarRespostas(campos, valores)`) compartilhada
  client/server — o client dá feedback, mas a **autoridade é o servidor** (revalida
  na submissão a partir da mesma definição).
- **as respostas não moram no formulário**: moram junto do envio que gerou o token
  (a avaliação de experiência, o destinatário da campanha), guardadas em `jsonb`
  pela chave do campo. O formulário é só a definição — o mesmo serve vários envios.

**A quarta peça: a APURAÇÃO** (ago/2026). O mesmo `tipo` que diz como desenhar o
campo diz como resumi-lo — marcação vira contagem por opção, escala vira
distribuição + média + top-2-box, número vira média/mediana/faixas, texto não se
resume e se lista. Com isso o painel de resultados se monta sozinho: pergunta nova
no construtor já nasce com o seu bloco, sem caso no código por pergunta. O
**recorte** também sai da definição — toda marcação de uma opção com poucas opções
(setor, tempo de casa) vira filtro, sem ninguém marcar "campo de segmento".

Três armadilhas da conta, todas vindas de o dado ser genérico:

- o valor de uma escala guardado é o **índice** (0..n−1); a média tem de sair na
  escala que a pessoa vê (1..n), e índice fora da escala (pergunta editada depois
  da resposta) se descarta em vez de contaminar;
- **percentual é sobre quem respondeu a pergunta**, não sobre o total da rodada —
  senão pergunta opcional aparece pior do que é;
- ao juntar perguntas de escalas diferentes num índice único, **normalize cada uma
  para 0..100 antes de somar**: sem isso, uma escala de 4 níveis pesa diferente de
  uma de 5 para a mesma opinião.

Ao salvar os campos: **upsert + poda preservando o id** (atualiza os que têm id,
insere os novos, apaga os que sumiram) em vez de apagar-e-recriar — assim editar um
formulário não invalida respostas históricas guardadas por id de campo.

## O que mais vale lembrar

- `tipo` fechado no `check`, o resto no `config` jsonb: o eixo estável no schema, o
  variável no dado.
- O renderer, a validação e a apuração leem a MESMA definição — não dá pra a tela,
  o servidor e o painel divergirem sobre o que o formulário pede.
- A apuração é **pura** (sem servidor, sem banco): o painel recebe as respostas uma
  vez e recalcula o recorte no cliente, então trocar de segmento é instantâneo e a
  conta se testa sem subir nada.

## Conexões
- Princípio: [[A definição em dado dirige o comportamento, não um caso no código]]
- Irmã: [[Formulário público por token opaco fica fora do gate de sessão]]
- Irmã: [[Recorte pequeno em pesquisa anônima identifica, então o painel se recusa a mostrar]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Dados]]
