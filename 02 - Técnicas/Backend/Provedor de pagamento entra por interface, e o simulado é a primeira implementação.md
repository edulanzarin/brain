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

## O que mais vale lembrar

- O nome do provedor é gravado no pedido (`provedor: "simulado"`), junto da referência
  externa. Sem isso, o dia em que existirem dois provedores em produção, o registro não
  diz por onde o dinheiro passou.
- Provedor real traz duas coisas que o simulado não tem: latência e resposta assíncrona.
  O estado `PENDENTE` já existe justamente para isso — o simulado só passa por ele
  depressa.
- Fronteira parecida com a de qualquer serviço externo: quem chama fora do processo
  precisa tratar erro e demora, não supor sucesso.

## Conexões
- Princípio: [[Chamada externa tem timeout e erro tratado]]
- Irmã: [[Acesso comprado é linha própria, não status do pedido]]
- Visto em: [[monofire]]
- Mapa: [[Backend]]
