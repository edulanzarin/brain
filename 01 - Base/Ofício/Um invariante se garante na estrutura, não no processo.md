---
tags: [tipo/atomica, camada/principio, dev/backend]
criado: 2026-07-28
---

# Um invariante se garante na estrutura, não no processo

> Quando a regra do domínio diz "existe no máximo uma X", faça a **estrutura do
> dado** recusar a segunda — uma linha única, um recurso compartilhado, uma
> credencial consumível — em vez de confiar que o **processo** (as pessoas, a
> ordem das chamadas, o front) vai se comportar. Processo falha em silêncio;
> estrutura recusa na cara.

## O problema

O caminho natural é modelar o invariante como uma expectativa do fluxo: "só um
gestor vai responder", "a tela não deixa clicar duas vezes", "o job roda uma vez
por dia". Cada uma é uma promessa que o mundo quebra — dois cliques concorrentes,
duas pessoas com o mesmo link, o job reexecutado. O resultado não é um erro
barulhento: é dado duplicado ou ambíguo que só aparece semanas depois.

## A solução

Empurrar o invariante para o lugar onde ele é uma **impossibilidade**, não uma
gentileza:

- "No máximo uma resposta por recurso" → `unique` + `insert ... on conflict do
  nothing`. A segunda tentativa não grava, ponto — não importa quem chamou.
- "Uma resposta canônica de um grupo" → uma credencial **compartilhada** pelo
  grupo, o primeiro fecha; não uma por membro esperando que só um use. Ver
  [[Uma resposta canônica de um grupo é um token compartilhado]].
- "Este colaborador não se repete neste envio" → índice único parcial, não um
  `if (jaExiste)` que corre contra concorrência.

## O que mais vale lembrar

Vale para o invariante de **existência/unicidade**. É primo de
[[Permissão se valida no servidor, não na interface]]: as duas dizem "não confie
no processo/na UI, garanta no lugar estrutural" — lá o lugar é o servidor, aqui é
o schema. E é o outro lado de [[A definição em dado dirige o comportamento, não um caso no código]]: se o comportamento mora no dado, a regra dura também.

## Onde já apareceu (dois casos, mesma lição)

- **Idempotência da submissão pública** do Nexo: o formulário respondido uma vez
  recusa o resto por `unique(experiencia_id)` + `on conflict do nothing`, não por
  a tela esconder o botão. Ver [[Formulário público por token opaco fica fora do gate de sessão]].
- **Avaliação sobre colaborador**: os gestores de um setor dividem UM token; o
  primeiro que responde fecha (os outros veem "já respondido"), e um índice único
  impede o mesmo colaborador duas vezes no envio — nada disso depende de o
  processo se comportar.

## "Exatamente uma destas duas colunas"

O ORM costuma não saber dizer isso, e a saída fácil é deixar a garantia na
disciplina de quem escreve. No [[Privello]], um pagamento pertence a uma
assinatura de plano OU a uma assinatura de perfil, nunca às duas e nunca a
nenhuma. Duas tabelas de pagamento resolveriam sem CHECK e duplicariam a
fronteira do provedor — que é justamente o que não pode ter duas versões.

A resposta é escrever o CHECK à mão na migration:

```sql
ALTER TABLE "pagamentos"
  ADD CONSTRAINT "pagamento_pertence_a_uma_compra"
  CHECK (("assinaturaId" IS NULL) <> ("assinaturaDePerfilId" IS NULL));
```

O `<>` entre dois booleanos é o XOR, e ele recusa as duas pontas de uma vez: o
pagamento órfão e o pagamento de duas compras deixam de ser representáveis. A
migration gerada não traz isso — editá-la é parte do trabalho, não um desvio
dele.

## Conexões
- Irmã: [[Permissão se valida no servidor, não na interface]]
- Depende de: [[A definição em dado dirige o comportamento, não um caso no código]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Base]]
