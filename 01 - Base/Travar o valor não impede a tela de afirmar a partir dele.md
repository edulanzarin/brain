---
tags: [tipo/atomica, camada/principio, armadilha]
criado: 2026-08-22
---

# Travar o valor não impede a tela de afirmar a partir dele

> Quando um cálculo devolve absurdo, o reflexo é **travar o número** — `clamp`, `max`,
> `min` — e considerar o assunto encerrado. Mas o valor travado continua descendo pelo
> sistema, e tudo o que foi derivado dele continua sendo dito com a mesma confiança de
> antes. A trava conserta o que se **lê**; não conserta o que se **conclui**.

## A regra

Ao travar um valor por ele estar fora do domínio, marque também que a leitura é
**inválida**, e faça cada consumidor decidir o que fazer com isso. Perguntas que os
consumidores precisam responder, e que a trava não responde por eles:

- **Projeção**: ainda faz sentido projetar? Quase nunca — projetar a partir de lixo
  travado devolve um resultado de aparência normal.
- **Cor**: o valor travado costuma cair no extremo da escada de cor, que quase sempre é
  o extremo *bom*. Cor também afirma.
- **Conselho**: recomendação derivada some ou muda? Conselho tirado de dado furado não é
  conselho conservador, é conselho errado.
- **Agregado**: o valor travado entra em soma, média ou total? Aí ele contamina algo que
  ninguém liga à leitura original.
- **Gravação**: o valor travado pode ser SALVO? Este é o pior, porque ele sobrevive à
  sessão em que o aviso estava na tela. No [[piwdex2]], o cadastro de pokémon do Stadium
  salvava a carta mesmo com a leitura impossível — e gravava IV 32 nos seis stats, que
  ninguém digitou. O estrago nem ficava ali: o Breeding lê esse mesmo campo como o IV do
  PAI, então um nível digitado errado numa ferramenta virava projeção de ovo mentindo na
  outra, sem nada ligando as duas. Leitura inválida não persiste.

Ausência de dado tem representação própria — um travessão — e ela precisa vencer todas
as escadas visuais, não entrar como mais um caso delas.

## Por que

O valor travado é **plausível por construção**: está dentro da faixa, formata bonito,
não dispara alarme nenhum. É exatamente por isso que ele engana melhor que o absurdo
original.

Caso concreto: uma calculadora estimava IV invertendo a fórmula de stats. Com a Quality
digitada errada, a estimativa saía fora de 0–32. A correção travou as duas pontas e
resolveu o número. Só que os IVs travados viravam 32 nos seis stats, e disso saía, tudo
na mesma tela e logo abaixo de um aviso vermelho dizendo pra não acreditar na leitura:

- a projeção ficava idêntica à do indivíduo perfeito, com os seis medidores dizendo "no
  teto" em **verde**;
- o travessão de ausência caía no ramo `valor >= máximo` da escada de cor e saía na cor
  de valor perfeito;
- uma nota escrevia por extenso que os seis stats já estavam no máximo, aconselhando não
  gastar um item que de fato ajudaria.

A trava tinha sido feita. Nenhuma das três afirmações mudou.

## Na prática

O mesmo padrão aparece longe da tela, sempre que um objeto é reaproveitado com um campo
corrigido e outro herdado: um catálogo que revalidava o frescor e devolvia
`{ ...anterior, checado: agora }` continuava carregando o `erro` e o `ao vivo: false` de
uma falha antiga — o dado tinha sido atualizado, a **conclusão sobre ele** não.

Regra prática: quando corrigir um campo, procure quem **deriva** dele antes de fechar.
Corrigir sem revisar as derivações troca um erro visível por um erro convincente.

## Conexões
- Irmã: [[A tela não afirma mais precisão do que a fonte tem]] ·
  [[Zero na tela é afirmação, não valor de conforto]] ·
  [[Estimativa que inverte valor arredondado é faixa, não ponto]]
- Visto em: [[piwdex2]]
- Mapa: [[Base]]
