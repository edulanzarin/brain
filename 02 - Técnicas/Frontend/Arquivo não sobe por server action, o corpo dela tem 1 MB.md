---
tags: [tipo/atomica, camada/padrao, armadilha, dev/frontend]
criado: 2026-08-30
---

# Arquivo não sobe por server action, o corpo dela tem 1 MB

> `Body exceeded 1 MB limit`. A mensagem vem do Next ANTES de qualquer código
> seu rodar, então a validação de tamanho que você escreveu com carinho nunca é
> alcançada — e foto de celular estoura o teto sozinha.

## O problema

Formulário com `<input type="file">` e `action={acaoDeServidor}` é o caminho
natural no App Router, e funciona lindamente enquanto o teste é feito com um
PNG de 70 bytes. No primeiro arquivo de verdade, erro de runtime na cara.

A saída óbvia é subir o teto:

```ts
// next.config.ts — resolve o sintoma no lugar errado
experimental: { serverActions: { bodySizeLimit: "64mb" } }
```

Isso abre **todas** as ações do sistema para corpos de 64 MB, e o corpo de uma
action é lido inteiro na memória. Um limite que existia para proteger o
servidor vira um limite que não protege mais nada, por causa de um formulário.

## A solução

**Bytes por rota, mutação por ação.** Route handler lê o corpo sem esse teto, e
o resto — apagar, marcar, renomear — continua ação, porque ali o corpo é um id.

```ts
// app/api/midia/route.ts
export async function POST(pedido: Request) {
    const corpo = await pedido.formData();
    const arquivo = corpo.get("arquivo");
    // o dono sai da SESSÃO, nunca de um id no corpo
}
```

Do lado da tela, `fetch` num laço, **um arquivo por chamada**. Não é detalhe de
implementação, é o que dá três coisas de graça: a tela conta "2 de 5" enquanto
sobe, o erro do quinto não joga fora os quatro que entraram, e o servidor nunca
segura cinco vídeos na memória ao mesmo tempo. Depois do laço, `router.refresh()`
— a grade é do servidor e sem isso a foto só apareceria na próxima visita.

## A armadilha que vem junto: `input.files` é referência viva

Para permitir reescolher o MESMO arquivo depois de um erro, é preciso zerar o
`value` do campo — sem isso o `change` não dispara de novo. Só que zerar o
`value` **esvazia `input.files`**, e quem guardou a lista guardou uma referência
para ela, não uma cópia:

```ts
const escolhidos = e.currentTarget.files;  // referência viva
e.currentTarget.value = "";                // esvaziou a lista acima também
if (escolhidos.length) enviar(escolhidos); // nunca entra
```

O envio some sem erro, sem log e sem requisição na aba de rede — não há nada
para procurar, porque nada aconteceu. Copiar antes resolve:
`const escolhidos = [...(e.currentTarget.files ?? [])]`.

## A regra que fica

A função que faz o trabalho mora **fora** do módulo `"use server"`. Com a
diretiva, todo export vira endereço público chamável de fora, e uma função de
regra não precisa dessa porta. A rota e as ações importam do mesmo módulo
comum, e a conferência de dono acontece uma vez, lá dentro.

## Conexões
- Folha isolada por enquanto: falta o princípio de "cada mecanismo tem uma
  classe de carga, e forçar a errada custa o limite que protegia o resto".
- Irmã: [[A regra que a server action executa mora fora dela]] ·
  [[React reseta o formulário ao fim de uma Server Action]]
- Visto em: [[Privello]]
- Mapa: [[Frontend]]
