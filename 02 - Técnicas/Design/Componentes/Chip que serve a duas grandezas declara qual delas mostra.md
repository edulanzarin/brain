---
tags: [tipo/atomica, camada/padrao, design, armadilha]
criado: 2026-08-19
---

# Chip que serve a duas grandezas declara qual delas mostra

> Duas medidas diferentes que compartilham os nomes dos degraus (comum, raro, lendário)
> e desenham o mesmo chip viram, na prática, uma só na cabeça de quem lê. Aí a mesma
> entidade aparece "comum" numa tela e "lendária" na outra, e o produto parece quebrado.

## O problema

Não é bug de dado: as duas telas estão certas, cada uma no seu eixo — a espécie é comum, o
indivíduo é lendário. O erro é de forma. O componente aceitava um valor já convertido, então
o eixo ficava só na cabeça de quem escolhia o componente, e escolher errado não dava
nenhum sinal: compila, renderiza, fica bonito e mente.

## A solução

O eixo entra no componente, não na disciplina de quem usa:

- **Um componente por eixo** (`RarityBadge` da espécie, `QualityBadge` do indivíduo), os
  dois delegando ao mesmo primitivo interno — cor e forma continuam únicas.
- O primitivo **exige saber de qual eixo fala** (`kind: "species" | "individual"`), e usa
  isso para se identificar: título no hover, ícone ou prefixo. Campo obrigatório é o que
  transforma "lembrar" em "não compila".
- **Regra escrita no arquivo do componente**, não numa conversa: "tela de indivíduo usa
  este; catálogo usa aquele".
- Onde os dois aparecem juntos, um deles ganha marca visível — cor igual só se pode
  quando o significado é igual.

## O que mais vale lembrar

Suspeite sempre que duas grandezas dividirem vocabulário. Espécie x indivíduo, plano x
consumo, previsto x realizado, bruto x líquido: o usuário não carrega o dicionário, ele lê
a palavra que está na tela.

## Visto em

No piwdex a Conta mostrava a raridade da ESPÉCIE no time (todo mundo "comum") enquanto o
painel, o modal de stats e o próprio jogo mostravam a faixa do INDIVÍDUO derivada da
quality (o mesmo Abra, "lendária"). O arquivo de raridade já avisava dos dois eixos — o
aviso não impediu o erro; o parâmetro obrigatório impede.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[Primitiva de botão fecha o tamanho e abre só a variante]]
- Visto em: [[piwdex]]
- Mapa: [[Design]]
