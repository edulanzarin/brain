---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-30
---

# Rótulo feito de chave técnica aponta para o registro errado quando os dois ids se parecem

> Um mesmo registro costuma ter **dois identificadores**: a chave interna do sistema e o número que o mundo usa (nota fiscal, pedido, contrato). Quando os dois têm a mesma forma — inteiros curtos, o mesmo nome coloquial —, um rótulo humano montado a partir da chave (`Nota 18675`) não parece errado: parece uma referência válida a **outro** registro, que existe. O log fica plausível e falso ao mesmo tempo.

## O caso

A rota de drill-down recebe `?chave=18675` (a `chavelctofissai` do Questor) e
gravava na trilha `alvo: "Nota 18675 (saída)"`. Aquele documento tem `numeronf`
**211**. A nota 211 existe, a nota 18675 também — só que é outra. Quem lesse a
trilha numa investigação iria atrás do documento errado, sem nenhum sinal de que
o rótulo estava mentindo.

Não é erro de código: a rota funciona, o parâmetro é o certo, o dado volta
correto. O defeito é **de vocabulário**, e por isso passa por revisão e por
teste — nenhum dos dois lê o rótulo.

## A regra

Rótulo destinado a humano **nomeia o espaço de identificador que está usando**:

```ts
// mente: parece número de documento e não é
alvo: `Nota ${chave} (${lado})`
// não mente: diz de qual id se trata
alvo: `Chave ${chave} · ${lado}`
```

Duas saídas, nesta ordem de preferência:

1. **Qualificar o id** que você já tem na mão — "Chave", "id interno", "código".
   Custa uma palavra e resolve.
2. **Buscar o número humano** só se ele já estiver no caminho. Uma consulta extra
   ao cabeçalho *só para enfeitar um log* não se paga — o par (chave + empresa)
   já identifica sem ambiguidade quem for atrás.

## O que mais vale lembrar

- O cheiro é o **nome coloquial compartilhado**: se a equipe chama as duas
  colunas de "número da nota", o rótulo vai ser ambíguo por padrão.
- Vale para toda mensagem que sai do sistema para uma pessoa: log, trilha de
  auditoria, e-mail de erro, toast, título de modal, nome de arquivo exportado.
- Só se pega com **dado real**. Em base de teste os dois ids costumam ser
  pequenos e parecidos, e a troca não chama atenção — este aqui só apareceu ao
  rodar contra produção.

## Conexões
- Princípio: [[Tirar o dado errado não põe a verdade no lugar]] — rótulo plausível e falso é pior que rótulo técnico e feio
- Irmã: [[Campo cujo nome você não sabe se lê do payload, nunca se chuta]]
- Visto em: [[Navetech Hub]] — drill-down de nota nos módulos Fiscal e Contábil
- Mapa: [[Backend]]
