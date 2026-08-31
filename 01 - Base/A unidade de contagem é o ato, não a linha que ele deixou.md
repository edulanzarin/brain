---
tags: [tipo/atomica, camada/principio, dados, armadilha]
criado: 2026-08-31
---

# A unidade de contagem é o ato, não a linha que ele deixou

> Um gesto humano quase nunca grava uma linha: grava as que o modelo do sistema exigir. Fechar os encargos de uma empresa grava uma por funcionário; apurar um mês grava uma por imposto; ler uma página grava uma por recarga. Contar linhas mede o formato da tabela, não o trabalho — e o erro não parece erro, porque o número é grande, plausível e cresce quando o trabalho cresce.

## A regra

Antes de contar qualquer coisa, pergunte **quantas linhas um ato gera**. Se a
resposta não for "uma", a contagem tem de agrupar pela chave do ato, e o número
de linhas vira um dado secundário — informativo, nunca a unidade.

## Por que

Porque a distorção não é uniforme, e é isso que a torna perigosa: ela **reordena**
o ranking em vez de só inflar o total. Quem faz o trabalho que grava muitas
linhas sobe; quem faz o que grava uma desce. Dois casos medidos, no mesmo mês:

- Fechar encargos da folha: **18.504 linhas para 137 atos** (135×). Contando
  linha, era o maior trabalho do departamento — quando eram 137 fechamentos em
  101 empresas. Ao lado, 332 rescisões calculadas, cada uma mais cara que um
  fechamento, apareciam como 0,4% do mês.
- Apurar imposto: **2.227 linhas para 968 fechamentos** (2,3×), porque um
  fechamento grava uma linha por imposto — três tipos saíam sempre com contagem
  idêntica, o que é a assinatura do lote.

Somados os dois efeitos num mesmo painel, o mês do departamento eram 83.889
"registros" dos quais um único trabalho em lote fazia 83%. Com a contagem por
ato, 15.102 — e o topo do ranking mudou de pessoa.

O caso da web é o mesmo princípio com outra roupa: "mais visto" que conta
requisição é um placar de F5 —
[[Contador de popularidade conta votante, não evento]].

## Na prática

- **Ache a chave do lote na fonte.** Costuma existir e ter nome de processo:
  `codigopercalculo`, `compet`, `id_execucao`, `batch_id`. Quando existe, o
  agrupamento é ela mais a entidade.
- **Quando não existe, escolha o corte e diga qual foi.** Numa transmissão sem
  id de lote, medi três: por minuto (conta retentativa como ato novo), por dia
  (corresponde a "mandei isso hoje") e sem data (funde o mês num ato). Escolher
  é legítimo; esconder que houve escolha, não.
- **Mostre os dois números.** "137 fechamentos, 18.504 linhas de funcionário" diz
  mais que qualquer um dos dois sozinho, e mata a pergunta "esse número está
  certo?" antes de ela nascer.
- **Contagem distinta por pessoa não soma para o total.** Um lote tocado por
  duas pessoas conta uma vez para cada — certo no ranking, errado no total do
  time. O total sai de agregado próprio, não da soma do ranking.
- **O sinal de alerta é contagem idêntica entre categorias.** Três tipos com
  exatamente o mesmo número mês após mês não é coincidência: é um ato só,
  escrevendo três linhas.

## Conexões
- Irmã: [[Auditar o registro, não só o agregado]] — a lente oposta: lá o agregado esconde o registro anômalo; aqui o registro esconde o ato
- Irmã: [[Razão só afirma quando os dois lados vêm do mesmo trabalho]]
- Técnica que aplica: [[Contador de popularidade conta votante, não evento]]
- Visto em: [[Navetech Hub]] — Apuração do Fiscal e Produtividade do DP
- Mapa: [[Base]] · [[Dados]]
