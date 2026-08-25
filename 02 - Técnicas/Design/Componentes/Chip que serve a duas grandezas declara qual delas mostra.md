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

## A variante: uma escada só, com o eixo dito no texto ao lado

Há um caminho oposto que também fecha, e ele apareceu depois — vale registrar
porque a escolha entre os dois é de contexto, não de gosto.

Em vez de dois componentes, **uma escada só** (mesma cor, mesmo símbolo por
degrau) servindo os dois eixos, e o eixo declarado pela **frase colada nela**:
"Lendário · quality 1.82" é indivíduo; "Lendário" sozinho, na ficha da espécie, é
catálogo. Funciona sob duas condições, e as duas têm de ser verdade:

1. **Os dois eixos nunca aparecem lado a lado.** Cada tela fala de um sujeito só.
   Onde eles se encontrarem, volta a valer a regra de cima.
2. **A escada mais longa cabe inteira no primitivo.** Se um eixo tem degraus que o
   outro não tem, o primitivo precisa dos dois conjuntos. Faltando, o chamador é
   empurrado a emprestar o degrau vizinho — e emprestar afirma que os dois são o
   mesmo, que é o erro original de volta pela porta dos fundos.

O ganho é aprendizado: quem decorou o brasão na grade do catálogo reconhece o
mesmo brasão na calculadora sem reaprender. O risco continua sendo o da nota — se
as duas telas se encostarem um dia, a fusão volta.

## O que mais vale lembrar

Suspeite sempre que duas grandezas dividirem vocabulário. Espécie x indivíduo, plano x
consumo, previsto x realizado, bruto x líquido: o usuário não carrega o dicionário, ele lê
a palavra que está na tela.

## Visto em

No piwdex a Conta mostrava a raridade da ESPÉCIE no time (todo mundo "comum") enquanto o
painel, o modal de stats e o próprio jogo mostravam a faixa do INDIVÍDUO derivada da
quality (o mesmo Abra, "lendária"). O arquivo de raridade já avisava dos dois eixos — o
aviso não impediu o erro; o parâmetro obrigatório impede.

Depois, no mesmo produto, a variante de cima: o brasão de raridade — um desenho por
degrau, com a escada na forma e não só na cor — passou a servir os dois eixos, e teve
de crescer de seis degraus (a raridade da espécie) para nove (a quality do indivíduo,
que tem `WEAK` embaixo e `ANCIENT`/`DIVINE` em cima). Foi a condição 2 cobrando o
preço dela antes mesmo de a decisão ir pro ar.

No piwdex2 a mesma armadilha voltou noutro eixo: "valor" do pokémon caía de `sellValue`
(o que o jogo paga por abate) pra `priceNpc` (preço do cassino) quando o primeiro era
zero. Os dois números moravam no mesmo campo, com o mesmo rótulo — e um ranking de
"paga mais por abate" coroou o Aerodactyl com 6,5 bilhões, que é o que ele CUSTA, sendo
que ele nem se caça. A correção foi a mesma: o campo passou a carregar uma bandeira
(`valueFromNpc`) e o rótulo muda com ela.

Terceira aparição, agora numa TABELA e não num componente: a lista de itens tinha as
colunas "melhor fonte" e "nível mínimo" lado a lado, e cada uma falava de uma espécie
diferente — a melhor fonte de Wool Ball é um Zangoose de nível 470, o nível mínimo é de
um Meowth de 20. Lidas na mesma linha, as duas afirmavam um "Zangoose nível 20" que não
existe. A regra é a mesma, e o custo de aplicá-la também: o sujeito entra na célula
("Zangoose · nv 470"), em vez de morar no cabeçalho da coluna vizinha. **Colunas vizinhas
afirmam falar da mesma coisa** — quando não falam, cada uma diz de quem fala.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[Primitiva de botão fecha o tamanho e abre só a variante]]
- Visto em: [[piwdex]] · [[piwdex2]]
- Mapa: [[Design]]
