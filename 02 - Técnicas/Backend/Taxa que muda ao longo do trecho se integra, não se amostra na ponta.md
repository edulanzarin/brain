---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-23
---

# Taxa que muda ao longo do trecho se integra, não se amostra na ponta

> Dividir o custo de um trecho inteiro pela taxa de UM ponto dele cobra o trecho todo
> naquele ponto. Quando a taxa melhora ao longo do caminho, amostrar a ponta produz uma
> estimativa **sempre otimista** — e sempre otimista pelo mesmo lado é viés, não ruído.

## O problema

O trecho é agrupado por conveniência de leitura (uma faixa de níveis, um lote, uma etapa),
mas a taxa dentro dele não é constante. Guardar só uma amostra e dividir por ela é o
atalho óbvio, e a amostra que sobra costuma ser a última — porque o laço termina nela.

No [[piwdex2]], a rota de treino agrupava níveis em faixas e guardava a estimativa do
último nível de cada faixa. Numa faixa 352→500 o mesmo alvo rende **11,7M de XP/h no
começo e 13,6M no fim**: a subida inteira estava sendo cobrada no passo mais rápido dela.
A rota prometia 99h28 onde a conta honesta dá 104h43. O mesmo viés ia junto para o ouro do
trecho, que saía das mesmas horas.

Nenhum caso de teste pega isso: a fórmula está certa, a taxa está certa, o resultado é
plausível. O erro só aparece contra a realidade, depois de alguém cumprir a rota.

## A solução

Somar dentro do laço que já calcula a estimativa de cada passo. O custo por passo já está
ali; só falta não jogá-lo fora:

```
para cada passo do trecho:
    custo   = custoDoPasso(passo)          // fórmula fechada, se houver
    horas  += custo / taxa(passo)
    saída  += (custo / taxa(passo)) * rendimento(passo)
```

E então **exibir a média do trecho, não a ponta dele**: `custoTotal / horas`. A ponta
continua útil para o que muda pouco (que golpe se usa, qual o risco); para rendimento ela
mente.

## O somatório tem que bater com a fórmula fechada

Trocar um total fechado por uma soma de partes cria uma invariante de graça, e ela é o
teste: **a soma das partes tem que dar exatamente o total fechado**. No piwdex2 essa
igualdade pegou dois defeitos que nenhuma inspeção tinha pego:

- **O trecho cobrava um nível a mais.** A conta era `xpTotal(fim + 1) - xpTotal(início)`,
  quando chegar ao nível alvo é parar de farmar nele. Um nível inteiro de XP a mais, e no
  fim da curva um nível não é pouco.
- **Nível sem alvo nenhum sumia da conta.** Um Psyduck de nível 1 não tem golpe que
  alcance ninguém, então aquele nível não entrava em faixa alguma — e o XP dele
  desaparecia, barateando a rota em silêncio. Passou a ser herdado pela primeira faixa,
  que estica o início até ele e se marca incompleta: o total vira "—" com aviso, em vez de
  sair menor do que é.

A varredura do catálogo inteiro (7.628 faixas, quatro percursos por espécie) conferindo
soma contra curva fechada fecha em zero divergências. Invariante que se checa em massa
vale mais que caso de teste escolhido a dedo.

## O que mais vale lembrar

- **Descubra a direção do viés antes de decidir se importa.** Taxa que melhora ao longo do
  trecho (você fica mais forte, o cache esquenta, o lote entra em regime) faz a ponta
  subestimar o tempo. Taxa que piora (a fila cresce, a tabela incha) faz superestimar.
  Otimismo sistemático em estimativa de prazo é o pior dos dois.
- **Trecho é agrupamento de leitura, não unidade de conta.** Agrupar por faixa deixa a
  tela limpa; a aritmética continua por passo. Quando os dois se confundem, o tamanho do
  agrupamento passa a mudar o resultado — e ninguém escolheu o agrupamento pensando nisso.
- Fora de jogo é o mesmo desenho: ETA de job cuja vazão muda ao longo da execução, barra
  de progresso que extrapola pelo último segundo, "no ritmo atual falta X", amortização em
  que a parcela de juros cai a cada mês.

## Conexões
- Princípio: [[Rendimento é vazão vezes tempo em pé, não vazão de pico]]
- Irmã: [[Total acumulado premia a lentidão quando o tempo é livre]] ·
  [[Limiar em grandeza contínua vira degrau, e o degrau decide a ordem]]
- Visto em: [[piwdex2]]
- Mapa: [[Backend]]
