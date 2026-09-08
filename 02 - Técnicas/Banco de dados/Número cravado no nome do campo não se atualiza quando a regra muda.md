---
tags: [tipo/atomica, camada/padrao, dados, armadilha]
criado: 2026-09-08
---

# Número cravado no nome do campo não se atualiza quando a regra muda

> A coluna se chamava `penalty20Percent`. Em 1.215 dos 1.366 contratos ela vale
> exatamente 10% do saldo, e nos outros 151 vale zero. Vinte por cento, nenhuma
> vez. Quem soma essa coluna e escreve "Multa 20%" no cabeçalho do relatório
> publica um número certo com um nome falso — e o nome é o que a diretoria lê.

## O problema

Nome de campo é documentação escrita **uma vez**, no dia em que o schema nasceu,
e nunca mais revisada. Quando a regra de negócio muda de 20% para 10%, alguém
altera o cálculo; ninguém renomeia a coluna, porque renomear quebra código,
migração e integração — e não renomear não quebra nada. O nome velho fica, e
fica plausível.

É pior que campo sem nome, pelo mesmo motivo do campo público que mente: **não
parece falta**. `penalty20Percent` é bem tipado, sempre preenchido, coerente com
os outros valores (`saldo - multa = a devolver` fecha em 1.176 dos 1.366 casos).
Nenhuma validação de schema pega, porque o schema está certo. O que está errado
é a frase que o nome afirma.

E o dano é específico do trabalho de relatório: o rótulo do cabeçalho quase
sempre sai do nome do campo. A mentira não fica no banco — ela é promovida a
título de coluna e apresentada como conclusão.

## A solução

Antes de somar um campo cujo nome afirma um número, **teste a afirmação contra a
coluna inteira**. É uma linha:

```js
const pct = {};
for (const l of leads) {
  if (!(l.remainingBalance > 0)) continue;
  const p = round2(l.penalty20Percent / l.remainingBalance * 100);
  pct[p] = (pct[p] || 0) + 1;
}
// { '10': 1215, '0': 151 }   -> o nome diz 20, e o dado diz 10 e 0
```

O que a distribuição responde, e o nome não:

- **Se há um valor dominante**, a regra existe e é essa — 10%, não 20%.
- **Se há um segundo grupo limpo** (151 zeros), não é erro: é uma segunda regra,
  uma isenção que alguém concede. Vale uma linha no relatório, não uma média.
- **Se o desvio fosse pulverizado**, não haveria regra nenhuma a nomear.

O teste da sistematicidade é o mesmo de
[[Config declarada envelhece; quem diz a regra é o comportamento observado]]:
quando 100% das ocorrências desviam do declarado, o errado é a régua.

## O que mais vale lembrar

- **Rotule pelo que o campo faz, não pelo que ele se chama.** No relatório a
  coluna virou "Multa retida", com a nota de que são 10% em 1.215 casos e zero
  em 151. O nome da origem não sobe pro cabeçalho.
- **Nomeie o seu próprio campo pelo papel, não pelo valor.** `penaltyAmount` ou
  `retainedFee` sobreviveriam à mudança de alíquota; `penalty20Percent` só
  sobreviveu porque ninguém olhou. Número no nome é constante mágica com
  publicidade.
- **Não corrija a fonte.** O schema é de quem cuida dele. Mostre a divergência —
  é achado do relatório, não bug a consertar por conta própria.
- Vale para qualquer nome que afirme regra: `limite30dias`, `taxa_15`,
  `desconto_black_friday`, `temp_`. O prefixo `temp_` é o caso extremo — é o
  nome mentindo sobre o tempo de vida em vez do valor.

## Conexões
- Princípio: [[Config declarada envelhece; quem diz a regra é o comportamento observado]]
- Irmã: [[Campo cujo nome você não sabe se lê do payload, nunca se chuta]] ·
  [[Campo que a normalização não copia vira número errado, não erro]] ·
  [[Auditar o registro, não só o agregado]] ·
  [[Rótulo feito de chave técnica aponta para o registro errado quando os dois ids se parecem]]
- Visto em: relatório de cancelamentos da UFit Academia (set/2026)
- Mapa: [[Dados]]
