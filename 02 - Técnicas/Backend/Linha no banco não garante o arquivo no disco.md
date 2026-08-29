---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-29
---

# Linha no banco não garante o arquivo no disco

> Banco e armazenamento são dois sistemas, e a referência de um para o outro é
> uma afirmação, não um invariante. Quem serve o arquivo confere que ele existe
> **antes** de abrir o stream — senão a resposta sai vazia, e resposta vazia não
> é erro nenhum na tela.

## O sintoma

```
$ curl -o /dev/null -w '%{http_code}' /midia/abc
000
curl: (52) Empty reply from server
```

Nem 404, nem 500: **nada**. O framework já tinha começado a responder 200 com os
headers quando o `createReadStream` encontrou o `ENOENT`, e não havia mais como
voltar atrás no status. O erro só existe no log do servidor. No navegador é uma
imagem quebrada sem explicação, e no monitoramento não é nada.

## A correção

```ts
if (!(await ARMAZENAMENTO.existe(arquivo))) {
  return new NextResponse("Indisponível", { status: 404 });
}
return new NextResponse(ARMAZENAMENTO.fluxo(arquivo), { headers });
```

Uma chamada de `stat` antes do stream. `existe()` entra na **interface** de
armazenamento, junto de `salvar` e `remover` — é pergunta de todo backend de
arquivo, não detalhe do disco local.

## Quando isso acontece de verdade

Não é caso teórico. As três formas que eu já vi:

1. **Banco restaurado sem a pasta.** Dump é fácil de copiar, volume de mídia
   não. O banco volta cheio de linhas apontando para o nada.
2. **Volume recriado.** `docker compose down -v`, ou volume nomeado trocado por
   bind mount, e o banco (que sobreviveu) continua descrevendo o que havia.
3. **Upload interrompido** entre gravar a linha e escrever os bytes.

O caso 1 é o comum, e é justamente quando alguém está debugando outra coisa e
não quer perder tempo com uma imagem quebrada sem mensagem.

## O que mais vale lembrar

- **Remover é idempotente**: apagar arquivo que já sumiu não pode derrubar a
  action. `unlink` dentro de `try/catch` vazio, de propósito.
- A checagem também dá o comportamento certo para a mídia que a moderação
  reprovou e teve o arquivo apagado, mas cuja linha ficou por histórico.

## Conexões
- Princípio: [[Chamada externa tem timeout e erro tratado]]
- Irmã: [[Servir anexo por rota com checagem de permissão]] · [[Trocar o backend de armazenamento sem downtime]]
- Visto em: [[Privello]]
- Mapa: [[Backend]]
