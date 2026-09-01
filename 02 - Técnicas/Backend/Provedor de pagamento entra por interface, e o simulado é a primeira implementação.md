---
tags: [tipo/atomica, camada/padrao, dev/backend]
criado: 2026-08-28
---

# Provedor de pagamento entra por interface, e o simulado é a primeira implementação

> O sistema não fala com Stripe nem com Mercado Pago: fala com um contrato de três
> campos. Trocar de gateway é escrever outro objeto, não reescrever a compra.

## O problema

Marketplace precisa cobrar, mas no primeiro dia não existe conta de gateway, chave de
sandbox nem decisão de qual usar. O caminho fácil é adiar a compra inteira ("depois eu
plugo o pagamento") e o resultado é pior: a regra de venda nasce colada ao SDK do
provedor que acabou sendo escolhido às pressas, e todo o fluxo — pedido, aprovação,
liberação de acesso — só se prova com credencial válida na mão.

## A solução

Uma interface pequena, que descreve o que a compra precisa do mundo externo, e nada mais:

```ts
export interface ProvedorPagamento {
  nome: string;
  cobrar(dados: CobrancaSolicitada): Promise<CobrancaResultado>;
}

export const provedorSimulado: ProvedorPagamento = {
  nome: "simulado",
  async cobrar({ pedidoId }) {
    return { aprovada: true, referenciaExterna: `sim_${pedidoId}` };
  },
};

export const PROVEDOR: ProvedorPagamento = provedorSimulado;
```

O resto do sistema importa `PROVEDOR`, nunca um SDK. O simulado aprova na hora e devolve
uma referência com cara de id de gateway — o suficiente para o fluxo de compra existir
inteiro, com pedido gravado, status e recibo, desde o primeiro dia.

Repare no que a interface **não** tem: nada de cartão, pix, boleto, webhook. Ela cobra e
responde. Quanto menos o contrato souber, menos ele muda quando o provedor real chegar.

## A interface é a metade fácil; a forma do fluxo é a que decide

O [[Privello]] teve a interface no papel e não teve nada disso no código: quatro
`provedor: "simulado"` escritos à mão, e a assinatura nascendo já valendo com o
pagamento gravado `PAGO` na mesma transação. O `PENDENTE` existia no enum e nada
o escrevia.

Isso passa como pronto porque funciona. E não é: com `PAGO` cravado na criação,
**todo o resto do sistema aprende que assinar e pagar são o mesmo momento** — a
tela que espera resposta na hora, o e-mail que sai junto, o teto que já conta o
plano novo. O adquirente de verdade não confirma no instante em que se pede, e
aí a integração não é trocar um objeto: é achar cada lugar que assumiu a
simultaneidade.

O teste é curto: **o caminho do simulado passa pelos mesmos estados que o do
provedor real, ou pula alguns?** Se pula, a interface é decoração.

## Ativar é porta própria, e ela precisa aguentar ser chamada duas vezes

A venda se parte em duas, e as duas existem desde o primeiro dia:

```ts
vender()              // cria a compra sem valer + pagamento PENDENTE, chama o provedor
confirmarPagamento()  // a ÚNICA porta que faz a compra passar a valer
```

Hoje quem chama a segunda é o simulado, na mesma requisição. Amanhã é o webhook,
e ela não muda — porque ela já não sabe de onde veio a confirmação. Idempotente
por construção, que é o que
[[Webhook de provedor chega repetido e fora de ordem, a borda tolera os dois]]
cobra: pagamento já confirmado devolve o mesmo desfecho sem escrever nada.

Tudo que é **consequência de ativar** mora ali dentro, não em quem vendeu: no
Privello, acender o destaque do anúncio. Deixado no lado da venda, o webhook
ativa a assinatura e esquece a bandeira.

E o prazo se calcula na **confirmação**, não na venda. Entre pedir e pagar pode
passar um dia, e quem renova antes de vencer tem o novo prazo empilhado sobre o
que ainda vale — contado na hora errada, ele soma sobre um saldo velho.

## Enquanto não há coluna de "confirmada"

A tentação é marcar a compra com uma bandeira dizendo que foi paga. Não precisa,
e é pior: quem responde se uma assinatura está de pé já era `fimEm > agora`.
A compra nasce com o **intervalo vazio** (`inicioEm == fimEm`), e confirmar é o
que lhe dá duração. Uma bandeira a mais seriam duas verdades sobre a mesma
pergunta, e toda leitura teria que consultar as duas para não errar —
[[Dado escrito por dois caminhos precisa de uma regra só, fora dos dois]].

## O que mais vale lembrar

- O nome do provedor é gravado no pedido (`provedor: "simulado"`), junto da referência
  externa. Sem isso, o dia em que existirem dois provedores em produção, o registro não
  diz por onde o dinheiro passou.
- Provedor real traz duas coisas que o simulado não tem: latência e resposta assíncrona.
  O estado `PENDENTE` existe justamente para isso, e o simulado **passa por ele**, só
  que depressa. Simulado que não passa é simulado que não ensaia nada.
- **Provedor que explode não deixa pagamento pendurado em PENDENTE.** Pendente quer
  dizer "esperando a pessoa", e ninguém está esperando um erro de rede: a exceção vira
  `FALHOU`.
- **Qual provedor atende vem do ambiente**, e nome desconhecido cai no simulado de
  propósito: em dev, um `.env` com erro de digitação não pode derrubar a compra inteira.
- Fronteira parecida com a de qualquer serviço externo: quem chama fora do processo
  precisa tratar erro e demora, não supor sucesso.

## Conexões
- Princípio: [[Chamada externa tem timeout e erro tratado]]
- Irmã: [[Acesso comprado é linha própria, não status do pedido]]
- Irmã: [[Webhook de provedor chega repetido e fora de ordem, a borda tolera os dois]]
- Visto em: [[monofire]] · [[Privello]] — no segundo, a interface estava na nota do
  projeto e não no código, e foi o que revelou que a metade que importa é a forma do
  fluxo.
- Mapa: [[Backend]]
