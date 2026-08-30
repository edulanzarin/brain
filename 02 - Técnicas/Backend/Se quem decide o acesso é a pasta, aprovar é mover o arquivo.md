---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-30
---

# Se quem decide o acesso é a pasta, aprovar é mover o arquivo

> Quando a separação entre privado e público é ESTRUTURAL — uma pasta tem rota
> aberta, a outra não tem rota nenhuma —, mudar o estado de um arquivo no banco
> não muda o acesso a ele. A linha diz "aprovado" e o endereço continua
> devolvendo 404, porque quem responde não é a linha.

## Onde isso morde

O vídeo de verificação chega numa pasta sem rota (`documento/`), espera a
moderação, e é aprovado. O aprovar parecia isto:

```ts
prisma.anuncio.update({ data: { video: comparacao.caminho } })
```

Compila, roda, grava. E o perfil passa a apontar para um arquivo que a rota
pública recusa por prefixo. O bug não aparece em teste de unidade nem em
typecheck: só aparece na tela, como uma imagem quebrada.

## A correção

A transição de estado inclui a **mudança de lugar**, e o caminho novo é o que
vai para o banco:

```ts
const publicado = await mover(caminho, `${PUBLICO}/${id}`);
if ("erro" in publicado) return { erro: publicado.erro };
// só agora o banco recebe o caminho, e ele é o de destino
```

Ordem importa: mover primeiro, gravar depois, e só então apagar a versão
anterior. Invertida, uma falha no meio deixa o registro apontando para um
arquivo que não está mais lá.

## Vale lembrar

O contrário também é regra: **recusar apaga o arquivo**. Ele nunca teve endereço
público e não vai ter, então guardá-lo é manter dado sensível de alguém no
servidor sem nenhum uso — e um dia alguém escreve a rota que o serve.

Se o projeto guarda "caminho de disco" numa coluna e "endereço público" em
outra, dê **nome à conversão** entre os dois em vez de escrevê-la à mão em cada
lugar. Foi exatamente onde este trabalho errou primeiro, servindo
`/midia/midia/publico/...`.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]] — a
  pasta É a permissão, e por isso a permissão só muda quando a pasta muda.
- Irmã: [[Servir anexo por rota com checagem de permissão]]
- Irmã: [[Linha no banco não garante o arquivo no disco]]
- Visto em: [[Privello]]
- Mapa: [[Backend]]
