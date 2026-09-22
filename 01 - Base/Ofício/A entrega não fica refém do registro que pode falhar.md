---
tags: [tipo/atomica, camada/principio]
criado: 2026-09-22
---

# A entrega não fica refém do registro que pode falhar

> Ferramenta que **entrega** alguma coisa (um resultado, um arquivo, um acesso) e
> de passagem **registra** alguma coisa (um lead, um log, uma métrica) tem duas
> obrigações de peso muito diferente. Encadear as duas transforma falha do lado
> secundário em falha total, e o usuário paga por um problema que não é dele.

## A regra

Ordene por quem depende de quem, e nunca o contrário:

1. **Grave o que precisa durar**, de forma síncrona, porque perder isso é
   irreversível.
2. **Entregue**, mesmo que o passo 3 não aconteça.
3. **Avise, espelhe, notifique** — tudo que é consequência vai solto, sem
   `await`, e a falha vira log.

Quando o registro é justamente a razão de existir da ferramenta (um muro de
cadastro antes do resultado), a ordem continua: tenta gravar, mostra o erro,
deixa tentar de novo. Mas depois de N falhas, **libera assim mesmo**.

## A troca, dita em voz alta

Liberar assim mesmo admite perder o registro. É a escolha certa quando o custo
de bloquear é maior que o de perder, e isso depende de quem está do outro lado:

| Do outro lado | Bloquear custa | Decisão |
|---|---|---|
| Público, uma vez só, sem vínculo | a pessoa vai embora e não volta | libera |
| Operador interno, em fluxo diário | ele tenta de novo daqui a pouco | bloqueia |

Um formulário de captação está sempre na primeira linha. Prender seiscentas
pessoas numa tela de erro porque o banco piscou custa a apresentação inteira;
perder alguns contatos custa alguns contatos.

## Na prática

No simulador tributário da Navecon o muro de cadastro fica entre o quiz e o
resultado. O envio é aguardado e o erro aparece, ao contrário da calculadora que
serviu de referência, onde o lead ia para uma planilha por `fetch` com
`mode: "no-cors"` — que não devolve resposta nenhuma, então um destino fora do ar
sumia com o contato sem ninguém saber. Mas a partir da segunda falha aparece um
"ver o resultado assim mesmo", e o quiz respondido não vira lixo.

Uma camada abaixo, o mesmo raciocínio: o lead é gravado no Postgres com `await`
e o e-mail de aviso sai com `void`. O e-mail é consequência do lead; o lead não
é consequência do e-mail.

## Por que

O erro que isto evita não é a indisponibilidade, é a **amplificação**. Um SMTP
lento, um webhook fora do ar, uma planilha que mudou de permissão são falhas
pequenas e comuns; encadeadas na frente da entrega, cada uma vira queda total da
ferramenta. A ordem certa converte cada uma de volta ao tamanho que ela tem.

## Conexões
- Irmã: [[Chamada externa tem timeout e erro tratado]] ·
  [[Contador que conta sucesso de promessa afirma que deu certo]] ·
  [[Laço que trata toda falha igual apaga a causa da primeira]]
- Visto em: [[Simulador Navecon]] · [[Evento Navecon]]
- Mapa: [[Base]]
