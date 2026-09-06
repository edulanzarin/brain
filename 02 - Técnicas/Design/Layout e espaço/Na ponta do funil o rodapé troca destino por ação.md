---
tags: [tipo/atomica, camada/padrao, design]
criado: 2026-09-05
---

# Na ponta do funil o rodapé troca destino por ação

> A barra de baixo do celular é o lugar mais caro da tela: uns 56px que o polegar
> alcança sem reposicionar a mão. Na lista ela vale como navegação. Na tela onde a
> pessoa decide, navegação ali é desperdício — quem chegou já navegou.

## O problema

O perfil do classificado é longo: retrato, ficha, valores, galeria, sobre,
avaliações e vizinhos. A fila de botões com o contato ficava no meio, e abaixo
dela havia uma dúzia de telas de rolagem em que o telefone não aparecia em lugar
nenhum. Quem leu tudo e decidiu chamar tinha que subir a página inteira de volta.

A saída óbvia é uma barra de contato presa no rodapé. Só que o rodapé já era da
barra de destinos, e aí aparecem três opções — as três com defeito, menos uma:

- **Empilhar as duas.** Somam uns 112px de uma tela que já é estreita, e cobrem
  justamente a foto que a pessoa foi ver.
- **Botar a ação como item da nav.** A barra de destinos afirma onde a pessoa
  está, acendendo um item. Um botão de ação no meio dela não tem lugar para
  acender, e a fileira passa a misturar "ir" com "fazer".
- **Trocar de dono.** Na rota do perfil, a nav sai e a barra de contato entra.

## A forma

Terceira opção. A rota decide quem é o dono do rodapé:

```tsx
// a nav conhece a rota e some onde não é dela
const noPerfil = /^\/[a-z]{2}\/[^/]+\/[^/]+\/?$/.test(rota);
<NavInferior className={noPerfil ? "max-md:hidden" : undefined} … />
```

Três coisas seguram a troca de pé:

- **A saída não pode depender só da barra.** No perfil ela continua no caminho
  (breadcrumb), no menu de seções grudado e no gesto de voltar do sistema. Sem
  esses três, tirar a nav seria prender a pessoa.
- **Só no telefone.** No monitor a fila de botões cabe acima da dobra, e uma faixa
  presa lá cobriria conteúdo sem ninguém ter pedido.
- **O que subiu para a barra sai da fila inline.** Repetir o mesmo botão em dois
  lugares na mesma tela não custa o botão: custa o que os dois empurram para fora.

## O que mais vale lembrar

O sinal para procurar essa troca é a razão entre a altura da página e a distância
até a ação. Página curta não precisa: a ação nunca sai de vista. O caso é a tela
longa que existe para produzir **uma** decisão — perfil, produto, artigo com
assinatura no fim.

## Visto em

No Privello, o perfil da acompanhante. A nav leva acompanhantes, favoritas,
anunciar e conta; no perfil o rodapé vira "chamar no WhatsApp" mais o coração.

## Conexões
- Princípio: [[O que responde pergunta rara não ocupa a rolagem de todo mundo]] —
  a mesma conta de orçamento de tela, aplicada ao rodapé fixo em vez da rolagem:
  quem ocupa precisa ganhar o lugar naquela tela, não no site inteiro.
- Irmã: [[Estado bloqueado aponta para a chave]] — a barra é onde o contato mora, e
  fechado ele leva ao passe em vez de ficar desligado.
- Visto em: [[Privello]]
- Mapa: [[Design]]
