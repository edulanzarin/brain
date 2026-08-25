---
tags: [tipo/atomica, camada/padrao, dev/frontend]
criado: 2026-08-25
---

# Sem servidor, a migração de dado local acontece na leitura

> Quando o dado mora no `localStorage` de quem usa o site, não existe rotina de virada
> possível: a única hora em que dá pra alcançar aquele navegador é quando ele abre a
> página. Então a migração vira parte da função que lê.

## O problema

Todo dado versionado uma hora muda de forma ou de lugar — a chave é renomeada, o campo
vira outro, a coleção sobe de dono. Com banco, isso é uma migration: roda uma vez, sobre
todas as linhas, antes do código novo subir.

Com `localStorage` não há "todas as linhas". Cada navegador tem a própria cópia, ninguém
consegue enumerá-las, e um site sem login não tem sequer a lista de quem existe. Publicar
o código novo com a chave nova e pronto significa uma coisa só: **todo mundo que já usava
o site abre a ferramenta e a coleção sumiu**. Sem erro, sem aviso, sem jeito de desfazer.

## A solução

A chave velha entra na função de leitura, **só de leitura**, e o resultado é somado ao da
chave nova:

```ts
const CHAVE = "app.bolsa.v1";
const CHAVE_ANTIGA = "app.estante.v1"; // nunca escrita

export function ler(): Item[] {
  const nova = lerChave(CHAVE);
  const antiga = lerChave(CHAVE_ANTIGA);
  if (!antiga.length) return nova;

  // Soma em vez de substituir: aba antiga em cache ainda escreve na chave velha,
  // e o que ela salvar tem de aparecer aqui.
  const vistos = new Set(nova.map((x) => x.id));
  return [...nova, ...antiga.filter((x) => !vistos.has(x.id))].slice(0, LIMITE);
}
```

A primeira gravação depois disso persiste tudo na chave nova, e a antiga vira inofensiva.

## O que mais vale lembrar

- **Somar, não substituir.** A aba velha em cache continua existindo e continua salvando
  na chave antiga. Se a leitura nova ignorar a velha assim que houver um item novo, o que
  aquela aba salvou some.
- **Não apagar a chave antiga.** O ganho é alguns bytes; a perda, se a leitura nova tiver
  um bug, é a coleção da pessoa. Deixe-a morrer sozinha.
- **A leitura já é desconfiada, e a antiga também.** O mesmo saneamento vale pras duas: o
  formato velho é, por definição, o que você já não controla mais.
- **A migração some sozinha.** Passado tempo suficiente, tirar a chave antiga da leitura
  é uma linha — e aí é decisão de quanto tempo você aceita perder alguém.

## Conexões
- Princípio: [[Estado mutável se lê da fonte no uso, não de cópia guardada]]
- Irmã: [[Migração de dados mantém o antigo como reserva até a virada]]
- Visto em: [[piwdex2]]
- Mapa: [[Frontend]]
