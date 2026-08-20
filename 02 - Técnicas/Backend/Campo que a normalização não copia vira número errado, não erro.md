---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-19
---

# Campo que a normalização não copia vira número errado, não erro

> A função que normaliza a fonte externa **é** o contrato do sistema com ela: o que não
> for copilado ali deixa de existir para o app inteiro. E como campo opcional costuma
> estar ausente na maioria dos registros, a perda não parece perda — o sistema continua
> rodando, só que com o número errado.

## O problema

Normalizar dado de terceiro é rotina saudável: preencher default, garantir shape, não
blindar cada acesso depois. O risco mora no formato dessa função — uma lista de campos
copiados um a um:

```ts
return {
  name: c.name,
  power: num(c.power),
  cooldownMs: num(c.cooldownMs),
  // ...e o campo que existe em 5% dos registros, que ninguém listou
};
```

Campo que aparece em poucos registros é exatamente o que passa despercebido na leitura
do JSON de exemplo — e é quase sempre o campo que **marca a exceção**, ou seja, o que
mais muda conta. No piwdex, `tm` (a marca de golpe de máquina) sumia assim. Resultado:
o motor tratava TM como golpe natural e prometia até **10x o DPS** que o jogador tinha,
em 164 das 482 espécies. Nada quebrou, nada logou, nenhum teste caiu — o Hunt Planner
só passou meses recomendando caçada com um golpe que o jogador não possui.

## A solução

- **Liste as chaves da fonte antes de escrever a normalização**, varrendo o conjunto
  todo, não o primeiro registro: `for (const x of todos) for (const k of Object.keys(x))`
  num Set. Leva um minuto e mostra o campo raro.
- Repita a varredura **depois de cada ingestão**: a fonte ganha campo sem avisar (foi
  assim que `area`, `captureBase`, `orreTier` apareceram).
- Quando o campo novo separa duas populações que o sistema tratava como uma, ele vira
  **parâmetro explícito** do motor, não um `if` escondido. Golpe natural x golpe de
  máquina virou um `pool` que atravessa o motor inteiro, com o padrão no caso mais
  conservador — o que todo jogador tem.
- Distribuição denuncia o campo perdido melhor que schema: contar `power` por valor
  mostrou 187 golpes de 600 num universo onde o resto vai até 200. Outlier em bloco não
  é acaso, é uma categoria que o modelo não conhece.

## O que mais vale lembrar

O tipo do TypeScript não protege aqui: o `as Creature[]` no fim do normalizador declara
que o dado está certo, e o campo continua chegando em runtime, invisível ao código.
Cast em fronteira externa é promessa, não verificação.

## Conexões
- Princípio: [[Auditar o registro, não só o agregado]]
- Irmã: [[Contador de terceiro conta no escopo dele, o seu recorte é delta sobre uma base]]
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
