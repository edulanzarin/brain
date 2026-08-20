---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-20
---

# Quando o campo numérico vem zerado, o número está na frase

> Payload de terceiro tem campo que o backend dele nunca preencheu, porque quem consome é
> a tela — e a tela lê o texto. Confiar no campo tipado devolve zero sem erro nenhum.

## O problema

O JSON parece desenhado para você:

```json
{ "key": "type-of-day", "name": "Tipo do Dia: Sombrio",
  "desc": "+20% de XP e +20% de loot em Pokémon do tipo Sombrio",
  "pct": 0, "until": 1787281200000 }
```

Existe um `pct`. Ele é número, tem o nome certo, e vale **zero** — enquanto a frase ao
lado diz 20%. O front do dono nunca usou o campo: ele renderiza `desc`. O campo é resto
de uma versão anterior, e nada no contrato avisa.

Ler `pct` não dá erro, não dá `null`, não dá exceção: dá um bônus de 0% que atravessa o
cálculo inteiro parecendo legítimo.

## A solução

Inverter a precedência: **a frase é a fonte primária, o campo é o plano B.**

```ts
const pct = pctPerto(desc, ["LOOT", "DROP"]) ?? (campo.pct > 0 ? campo.pct / 100 : null) ?? PADRAO;
```

Para extrair da frase sem escrever um parser frágil:

- **Case por proximidade, não por posição.** A mesma frase costuma trazer dois números
  ("+20% de XP e +20% de loot"); cada um pertence à palavra mais perto dele. Pegue todas
  as ocorrências de `N%`, ache a palavra-chave, escolha a de menor distância — isso já
  aguenta o número antes ("+40% de loot") e depois ("Loot Boost +40%").
- **Normalize antes de comparar**: acento fora (`NFD` + faixa de diacríticos), tudo em
  maiúscula. O texto vem no **idioma da conta**, não no seu.
- **Case palavra inteira** quando o alvo é um nome de uma lista fechada (tipo, categoria,
  status). Procurar substring faz "ACO" casar dentro de qualquer palavra; quebrar em
  tokens por não-letra e consultar um dicionário de apelidos resolve e ainda cobre
  português, inglês e espanhol de uma vez.
- **Nome desconhecido não vira palpite.** Se nenhum apelido casar, guarde o rótulo cru e
  mostre ele — a tela dizendo "Tipo do Dia: Gliscor" é honesta; escolher um tipo qualquer
  para preencher o campo não é.

## O que mais vale lembrar

- Um parser assim se testa com o payload **real** copiado do dia, mais variações de
  idioma e formato. É teste barato e é o único que pega o campo zerado.
- Se o array que você não conseguiu observar (nenhum boost ativo na hora) tiver formato
  diferente do imaginado, o parser tolerante devolve 0 — que é o caso comum — em vez de um
  número inventado. Errar para o lado do "sem bônus" é o lado seguro.

## Conexões
- Princípio: [[Chamada externa tem timeout e erro tratado]]
- Irmã: [[Número de regra alheia se lê da fonte, não se congela em constante]]
- Parente: [[Campo que a normalização não copia vira número errado, não erro]] · [[O bundle público do cliente entrega o contrato da API sem documentação]]
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
