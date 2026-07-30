---
tags: [tipo/atomica, camada/padrao, dev/backend]
criado: 2026-07-30
---

# De-para determinístico com override que vira aprendizado

> Casar a conta/código de um sistema com o de outro (plano de contas de origem →
> plano do ERP, CFOP → contabilização) parece pedir IA. Não pede. Uma **cascata
> determinística** — chave exata, depois estrutura, depois similaridade de texto
> restrita — resolve a maioria, marca o resto como "confira", e cada correção do
> humano vira um **override salvo** que o sistema reusa. A máquina propõe, a
> pessoa confirma o duvidoso, e o de-para fica melhor a cada uso.

## O problema

Dois planos de contas nunca têm a mesma numeração; descrições são parecidas, não
iguais. Pedir pra um LLM "adivinhar o de-para" é caro, não-reproduzível e erra em
silêncio. Mas cravar um `switch` no código também não escala: cada escritório,
cada CFOP, cada software de origem é um caso, e o caso muda.

## A solução

Resolver por **cascata**, do mais confiável ao menos, e deixar o resultado ser
**dado**, não código:

1. **Override salvo** — casamento que um humano já confirmou antes (confiança
   total).
2. **Estrutura** — mesma classificação hierárquica / mesma chave, quando existe e
   é única.
3. **Descrição normalizada** — maior similaridade de tokens (Dice), **restrita à
   mesma classe e natureza** pra não casar coisa aleatória. Acima de um limiar =
   casada; na faixa cinza = duvidosa (sugere, mas pede conferência).

O que não casou fica visível pro humano; a escolha dele **grava um override** e
alimenta o passo 1 da próxima vez. O de-para é uma tabela de DADOS que dirige o
comportamento — [[A definição em dado dirige o comportamento, não um caso no
código]] —, então evolui sem deploy.

Aparece duas vezes no mesmo sistema, em features diferentes: a contabilização por
CFOP (plano do Questor + override manual + aprendido do histórico) e a implantação
de saldos (plano de origem → `planoespec`). O mecanismo é o mesmo; muda só o que
está de cada lado.

## O que mais vale lembrar

A similaridade só é segura **com trava de contexto** (mesma classe, mesma
natureza): sem isso, "juros" casa com "juros" do grupo errado. O limiar tem duas
faixas de propósito — uma pra casar sozinho, outra pra sugerir e pedir olho
humano; um limiar só ou casa demais ou trabalha demais. E o override tem que ser
por escopo certo (por empresa, por estabelecimento) pra um aprendizado não vazar
pro caso vizinho.

## Conexões
- Princípio: [[A definição em dado dirige o comportamento, não um caso no código]]
- Irmã: [[Coleta determinística, LLM só interpreta]]
- Visto em: [[Navetech Hub]] (Implantação de Saldos; contabilização por CFOP)
- Mapa: [[Backend]]
