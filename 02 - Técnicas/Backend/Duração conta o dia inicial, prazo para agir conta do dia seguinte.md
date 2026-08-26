---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-26
---

# Duração conta o dia inicial, prazo para agir conta do dia seguinte

> `início + n` é a conta errada para duração. Um contrato de 45 dias que começa
> no dia 01 termina no dia 45, não no 46: o primeiro dia já é o dia 1, então a
> conta é `início + (n − 1)`. Prazo para praticar um ato é o outro caso — esse
> começa a correr no dia seguinte, e aí `início + n` está certo.

## O problema

As duas contagens usam a mesma palavra ("45 dias", "10 dias") e a mesma
aritmética de data, e por isso a diferença some no código. `addDias(inicio, n)`
parece a tradução óbvia de "n dias depois" — e é, quando o que se soma é um
**deslocamento**. Só que duração não é deslocamento: é um **intervalo fechado**
que já inclui o primeiro dia.

O erro nasce mudo. Ninguém recebe exceção, o teste do caso médio passa, a data
sai plausível — só um dia adiante. E um dia adiante num marco de experiência é
justamente o dia em que a decisão de prorrogar já não pode mais ser tomada.

Some ainda mais fácil quando o mesmo sistema tem os dois casos e eles estão
certos por acidente: no Nexo, o prazo de pagamento da rescisão conta do dia
seguinte à demissão (`data_dem + 10`) e a experiência conta a partir da admissão
(`admissão + 44`). Copiar a fórmula de um pro outro erra sem parecer que errou.

## A solução

Uma função só com a regra, e o resto do sistema chamando ela — não a aritmética
repetida em cada tela e em cada job:

```ts
/** O prazo começa NO dia da admissão (a admissão é o dia 1). */
export function vencimentoMarco(dataadm: string, marco: Marco): string {
  return addDias(dataadm, marco - 1);
}
```

Duas âncoras para não voltar a errar:

1. **Um exemplo concreto no comentário**, com mês de 30 e de 31 dias — quem
   entra 01/09 fecha 45 dias em 15/10; quem entra 01/08 fecha em 14/09. O
   exemplo é conferível de cabeça; a fórmula sozinha não é.
2. **Pergunte o que o número mede** antes de somar: duração de um estado
   (contrato, garantia, licença, período aquisitivo) inclui o dia inicial;
   janela para agir depois de um fato (pagar, recorrer, responder) começa no dia
   seguinte ao fato.

## O que mais vale lembrar

- **A data derivada também está gravada.** Se o vencimento foi materializado
  numa tabela (para o formulário público mostrar, para o cron comparar),
  corrigir a função não corrige o passado: vai junto uma migration recuando as
  linhas, e o upsert passa a atualizar a coluna em vez de só tocar
  `atualizado_em`. Senão a tela nova mostra a data velha.
- **A fórmula errada é conferível em dois pontos**, não em um: teste o primeiro
  dia e o último. `início + n` só se denuncia na borda.

## Conexões
- Princípio: folha isolada — nenhum princípio da Base cobre isso ainda; se a
  contagem inclusiva reaparecer noutro domínio, vira princípio.
- Parente: [[Número de regra alheia se lê da fonte, não se congela em constante]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Backend]]
