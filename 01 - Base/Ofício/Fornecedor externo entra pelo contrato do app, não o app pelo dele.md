---
tags: [tipo/atomica, camada/principio, dev/backend]
criado: 2026-09-17
---

# Fornecedor externo entra pelo contrato do app, não o app pelo dele

> Quando o sistema depende de uma empresa de fora — adquirente, mensageria,
> armazenamento, modelo de IA —, quem define a forma da conversa é o **app**. O
> fornecedor implementa essa forma. Deixar o formato dele vazar para dentro
> transforma trocar de fornecedor em reescrever o domínio.

## A regra

O núcleo declara o que precisa, em vocabulário do próprio negócio: "quero uma
cobrança Pix de N centavos", "quero mandar esta mensagem para este contato". Uma
peça fina por fornecedor traduz isso para a API dele, e **só ela** conhece nomes
como `transaction_amount`, `point_of_interaction` ou `message_sid`.

Três coisas ficam desse lado da costura, e é o esquecimento delas que faz a
abstração vazar depois:

1. **A tradução de vocabulário**, inclusive de unidade. O app fala em centavo
   inteiro; se a API fala em reais decimais, a conversão mora no adapter.
2. **A tradução de ESTADO.** `approved`, `charged_back`, `refunded` viram o
   punhado de situações que o domínio reconhece. Sem isso, cada `if` do sistema
   aprende o dicionário de um fornecedor.
3. **A verificação de autenticidade.** Cada um assina a notificação do seu jeito,
   e conferir isso é trabalho de quem conhece o formato —
   [[A assinatura autentica o dado, não quem o trouxe]].

## Por que

O motivo bom é a troca de fornecedor, mas ele é o menos frequente. O que se
ganha todo dia é outro: **dá para rodar o sistema inteiro sem o fornecedor
existir**. Enquanto a conta de adquirente não sai, o produto inteiro funciona com
um dublê, e o caminho de venda é percorrido de ponta a ponta muito antes de
haver dinheiro de verdade em jogo.

O custo de não seguir aparece quando o fornecedor muda de ideia, não quando você
muda de fornecedor. Preço, campo obrigatório novo, um estado a mais no
`enum` — se isso está espalhado, cada mudança deles vira uma caçada aqui.

O contrapeso: a costura só compensa quando o recurso é **trocável**. Banco de
dados escolhido, framework, linguagem — esses não se abstraem "por via das
dúvidas"; a camada extra ali é custo sem a troca que a pagaria.

## Na prática

- O adapter é uma interface pequena, com os verbos do negócio, não um espelho da
  API de ninguém.
- **Qual fornecedor está valendo é decisão do ambiente**, não do código que
  vende. E o padrão, quando nada é configurado, é o dublê — cair no fornecedor
  real por omissão faz a primeira venda de teste cobrar de verdade.
- O dublê tem que conseguir **fechar** o fluxo, não só começá-lo:
  [[Dublê que não fecha o fluxo deixa o caminho sem ninguém passar]].

## Onde já apareceu (três casos, mesma lição)

- **Mensageria** no navetalks e no navecrm: WhatsApp e e-mail atrás de uma
  costura só — [[Adapter de canal isola o app do provider de mensageria]].
- **Entrega de update sem IP público**: webhook e polling são dois fornecedores
  da mesma coisa, e o app não sabe qual está em pé —
  [[Polling substitui webhook quando não há IP público]].
- **Pagamento Pix** no [[telebot]] (set/2026): o motor do bot pede um Pix e
  recebe um Pix; Mercado Pago e o dublê simulado cumprem o mesmo contrato, e a
  plataforma rodou inteira meses antes de existir credencial.

## Conexões
- Irmã: [[A regra mora fora da porta que a chama]] · [[Configuração vem do ambiente, não do código]] · [[Chamada externa tem timeout e erro tratado]]
- Técnica que aplica: [[Adapter de canal isola o app do provider de mensageria]] · [[Webhook de dinheiro precisa de duas travas, a do evento e a do efeito]]
- Mapa: [[Base]] · [[Backend]]
