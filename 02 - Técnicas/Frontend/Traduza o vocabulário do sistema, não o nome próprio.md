---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-08-20
---

# Traduza o vocabulário do sistema, não o nome próprio

> Ferramenta em português sobre um sistema em inglês tem duas classes de palavra, e
> tratá-las igual quebra a tela de um jeito ou de outro: traduzir tudo arranca a ponte
> com o sistema original, não traduzir nada deixa o usuário lendo `COMMON` e `PHYSICAL`.

## O problema

O reflexo é decidir "traduz" ou "não traduz" para o arquivo inteiro. Os dois extremos
falham por motivos opostos:

- **Traduzir tudo** quebra o uso real. Quem está no piwdex procurando `Bulb` está com o
  inventário do jogo aberto na aba do lado. Se a dex chamar o item de "Bulbo", os dois
  textos deixam de casar e a ferramenta perde a única coisa que ela tinha de melhor que
  a memória do jogador.
- **Não traduzir nada** entrega `COMMON`, `PHYSICAL`, `GRASS` — vocabulário de schema,
  não de gente. Foi exatamente a reclamação que abriu isso: *"Commom como? Cada espécie
  tem comum, fraco, lendário"*.

## A solução

A linha não é por arquivo, é por **classe de palavra**:

| Classe | Exemplos | O que fazer |
|---|---|---|
| **Vocabulário do sistema** | tipo, raridade, categoria, estágio, papel, origem | **traduz** — é conceito, e conceito tem nome na língua do leitor |
| **Nome próprio** | espécie, item, golpe, mapa | **não traduz** — é a chave de busca compartilhada com o sistema original |

E o vocabulário traduzido mora num **único módulo de rótulos**, não espalhado na tela.
O mesmo tipo aparece no badge do card, no menu de filtro, no chip de filtro ativo, na
tabela de fraquezas e na ficha: traduzido em cinco lugares, um dia quatro concordam e um
discorda, e a tela passa a se contradizer sobre a mesma coisa.

Quando o próprio sistema tem locale, **use o nome dele**, não o seu: se o jogo chama o
tipo de "Sombrio" no idioma BR, a ferramenta chama de Sombrio — inventar "Noturno" cria
um terceiro vocabulário para o usuário decorar.

## A armadilha que a tradução cria: o acento na busca

Traduzir mexe no índice de busca, e isso morde duas vezes:

1. **O índice tem que carregar os dois idiomas.** A tela mostra "Fogo"; se o índice só
   tem `FIRE`, digitar o que está escrito na tela não acha nada.
2. **Acento tem que sair dos dois lados.** `"Psíquico"` só era encontrado digitando o
   acento — `psiquico` devolvia **zero** com 49 espécies na lista. Ninguém digita acento
   em caixa de busca.

```ts
const semAcento = (s: string) =>
  s.normalize("NFD").replace(/[̀-ͯ]/g, "").toLowerCase();

// normaliza o ÍNDICE e o que o usuário digitou — só um dos dois não resolve
haystack: semAcento(`${nome} ${id} ${tipoEn} ${tipoPt}`),
if (!e.haystack.includes(semAcento(busca))) return false;
```

## O que mais vale lembrar

O teste que pega o erro: **digite exatamente o que está escrito na tela**. Se o rótulo
diz "Psíquico" e digitar isso (com ou sem acento) não devolve a lista, o índice e a
interface estão falando línguas diferentes.

Vale para qualquer ferramenta sobre sistema alheio — ERP, API de terceiro, jogo. O nome
próprio é a chave estrangeira para o mundo real; o vocabulário é seu.

## Visto em

No piwdex2, sobre o catálogo (em inglês) do Poke Idle World.

## Conexões
- Princípio: [[Token semântico em vez de valor literal]]
- Irmã: [[Chip que serve a duas grandezas declara qual delas mostra]]
- Visto em: [[piwdex2]]
- Mapa: [[Frontend]] · [[Design]]
