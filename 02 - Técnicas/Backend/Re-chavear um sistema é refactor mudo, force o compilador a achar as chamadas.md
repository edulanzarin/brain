---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-24
---

# Re-chavear um sistema é refactor mudo, force o compilador a achar as chamadas

> Trocar por QUEM um sistema é chaveado — de usuário para conta, de conta para
> projeto, de empresa para filial — não muda um único tipo. Os dois ids são
> `string`. O compilador fica calado enquanto cada chamada continua compilando e
> passando a chave errada.

## O problema

A regra "um por usuário" costuma não estar no código: está na CHAVE PRIMÁRIA.
Enquanto `user_id` é a PK, a segunda linha é impossível de gravar — não é uma
validação que se afrouxa com um `if`, é a estrutura do banco. É por isso que
funciona tão bem, e é por isso que trocá-la é caro.

Trocada a chave, `lerConfig(id)`, `salvarDesejado(id, ...)`, `sessaoDe(id)`
continuam com a mesma assinatura. Passar o id do usuário onde agora se espera o
da conta compila, roda, e grava na chave errada — ou, pior, na conta de outra
pessoa. Num sistema de 27 arquivos, achar isso por leitura é apostar na atenção.

## A solução: mude a FORMA, não só o nome

Duas manobras baratas que transformam "eu preciso lembrar" em "o compilador me
mostra". Aplique-as nas funções mais chamadas e deixe o erro te guiar:

**Mude a aridade.** `registrarEvento(userId, ev)` virou
`registrarEvento(userId, contaId, ev)` — e o `TS2554: Expected 3 arguments, but
got 2` listou as cinco chamadas do motor de uma vez.

**Troque dois parâmetros por um objeto**, quando os dois são do mesmo tipo:

```ts
export interface DonoDaSessao {
  conta: string;    // o vínculo cuja credencial está aberta
  usuario: string;  // o assinante dono dele
}
segurar(quem: DonoDaSessao, tokens: Tokens, ...)
```

Aqui não era só ergonomia. Conta e usuário são ambos `string`: trocar a ordem
seria um bug **mudo** — a sessão abriria com a credencial certa e gravaria o
estado na chave errada. Com o objeto, `TS2345` apontou as duas chamadas.

## O portão novo que a chave antiga fazia de graça

Enquanto a chave era o usuário, "o registro do usuário logado" era a única linha
que existia: não havia como pedir a de outro. Com o id na URL, **toda leitura
por id precisa conferir o dono** — e o lugar de conferir é a própria consulta
(`WHERE id = $1 AND user_id = $2`), não um `if` depois de carregar. Assim não
existe caminho em que a credencial é lida antes de o dono ser checado.

Id que não é seu responde **404, não 403**: "existe mas não é sua" confirma a
existência do id para quem estava adivinhando.

## Do lado do cliente, o mesmo risco com outra cara

Se cada `fetch` monta a URL sozinho, esquecer o parâmetro em UMA chamada não dá
erro: ela cai no padrão do servidor e mostra o dado da outra chave **com cara de
certo** — o pior desfecho numa tela que existe para distinguir. Um único helper
(contexto + `useRota`) faz a URL ter um caminho só, e esse caminho nunca esquece.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[Ausência de leitura cai no valor que dispara a ação]] ·
  [[Escopo de dado se clampa no servidor, num funil só]] ·
  [[Cravar o seam de permissão antes do login]]
- Visto em: [[piwdex2]]
- Mapa: [[Backend]]
