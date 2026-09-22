---
tags: [tipo/atomica, camada/principio, armadilha]
criado: 2026-09-22
---

# Quando o degrau é real, preserve a monotonia em vez de suavizar

> [[Limiar em grandeza contínua vira degrau, e o degrau decide a ordem]] manda
> interpolar. Só que existe degrau que **é do mundo**, não da modelagem: a lei
> muda de regime no R$ 4,8 milhão e pronto. Ali não se suaviza nada. O que
> continua exigível é mais fraco e mais fundamental: **atravessar o degrau não
> pode inverter o sinal do total**.

## A regra

Antes de tentar tirar um degrau, pergunte de quem ele é.

- **Degrau da modelagem** (um `if x >= K` escolhido por conveniência para
  devolver um fator): interpole. É o caso da nota irmã.
- **Degrau do mundo** (faixa legal, tabela oficial, contrato): mantenha. Apagar
  o salto aqui é mentir sobre a realidade que a ferramenta descreve.

No segundo caso a garantia que sobra é a **monotonia**: se a entrada cresce, a
saída acumulada não pode diminuir. A empresa que fatura um real a mais não pode
aparecer pagando menos imposto no ano.

E monotonia não se confere lendo o código. Cada lado do degrau parece plausível
isolado; o que denuncia é varrer a entrada e comparar vizinhos. É teste de
propriedade, não de exemplo — o exemplo teria que cair exatamente em cima do
limiar para falhar, e ninguém escreve esse exemplo por acaso.

## Como o defeito entra

Quase sempre por **duas tabelas que descrevem o mesmo fato e discordam de
sinal**, cada uma escrita num dia diferente e nenhuma errada sozinha.

O caso concreto: no simulador tributário, uma tabela dizia que Lucro Presumido
com margem apertada é o **pior** enquadramento possível (nota 9 de ~20 na escala
de adequação). Outra tabela, a da carga, descontava pontos da alíquota quando a
margem era apertada — ou seja, afirmava que ali se paga **menos**. As duas
conviviam havia tempo sem sintoma.

A contradição tinha razão de ser óbvia depois de vista: no Presumido a base de
cálculo é uma margem **arbitrada em lei**, não a que a empresa teve. Margem
apertada significa pagar sobre lucro que não existiu, então a situação piora. O
desconto tinha o sinal trocado.

O sintoma só apareceu no cruzamento do teto do Simples, e só para um segmento em
que as duas cargas base empatavam: atravessar de R$ 400.000 para R$ 400.001 por
mês fazia o imposto anual **cair**, porque o Presumido que substituía o Simples
vinha "barateado" pela margem.

## Na prática

O conserto não foi suavizar nem mexer no limiar: foi pôr um piso no ajuste
(`{ peso: 0.4, minimo: 0 }`) para que no Presumido a margem **só agrave**. O
degrau continua lá, porque ele é verdade, e agora aponta para o lado certo.

O que trancou a correção foi um invariante e não um número esperado:

```ts
it("faturar mais nunca reduz o imposto estimado", () => {
  fc.assert(fc.property(arbRespostas, fc.integer({ min: 1 }), (r, delta) => {
    expect(calcular({ ...r, faturamentoMensal: r.faturamentoMensal + delta }).cargaAnual)
      .toBeGreaterThanOrEqual(calcular(r).cargaAnual);
  }));
});
```

Instância de [[Um invariante se garante na estrutura, não no processo]]: a
propriedade vale para toda combinação futura de tabela, inclusive as que ainda
vão ser recalibradas por outra pessoa.

## Por que

Ferramenta de estimativa não tem conserto depois de exibida. O número sai numa
tela, alguém decide em cima dele, e a inversão é justamente o defeito que
destrói credibilidade na hora: qualquer pessoa na plateia que brinque com o
valor de entrada encontra o ponto em que faturar mais paga menos, e a partir
dali nenhum outro número da tela vale nada.

## Conexões
- Irmã: [[Limiar em grandeza contínua vira degrau, e o degrau decide a ordem]] ·
  [[Calibre nas pontas, o meio esconde o defeito]] ·
  [[Fórmula verificada só vale na escala em que foi verificada]]
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Visto em: [[Simulador Navecon]]
- Mapa: [[Base]]
