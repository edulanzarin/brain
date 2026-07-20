---
tags: [tipo/atomica, projeto/navedesk, dev/backend, conceito]
criado: 2026-07-20
---

# Servir anexo por rota com checagem de permissão

> Arquivo enviado por usuário não vai para `public/`. Vai para um volume fora da
> web root, e é entregue por uma rota que confere a permissão **a cada request**.

## Por que não `public/`

Qualquer coisa em `public/` é servida crua, sem passar por código. Numa app
onde chamado é privado, isso significa que basta ter (ou adivinhar) a URL para
ler o print que outro setor anexou. "Ninguém vai adivinhar o nome" não é
controle de acesso.

## O desenho

1. **Disco, não banco.** Imagem em Postgres incha o dump e não ganha nada. O
   banco guarda só o metadado (nome original, mime, tamanho, quem enviou).
2. **Nome no disco é UUID gerado**, com a extensão derivada do mime. O nome
   original do usuário **nunca toca o filesystem** — é isso que fecha a porta
   para `../../etc/passwd`.
3. **Entrega por rota** `/api/anexos/[id]`: busca o metadado, resolve o chamado
   dono, aplica a mesma função de permissão do resto do sistema e só então faz
   stream do arquivo.
4. **Volume nomeado** no compose, para os anexos sobreviverem ao rebuild da
   imagem.

## Detalhes que valem lembrar

- Anexo pendurado em **nota interna** segue a visibilidade da nota, não a do
  chamado — senão a imagem do diagnóstico interno vaza pelo link direto.
- `Content-Disposition: inline` por padrão (imagem abre no navegador) e
  `attachment` com `?download=1`.
- `Cache-Control: private` — o navegador pode guardar, proxy não.
- `X-Content-Type-Options: nosniff`, para o navegador não reinterpretar um
  upload como HTML.
- Mesmo com o nome gerado, vale uma função `caminhoSeguro()` que recusa `/`,
  `\` e `..` — defesa em profundidade custa três linhas.

## Links

- Usado em: [[Navedesk]]
- Faz parte de: [[Desenvolvimento]]
- Ver também: [[Migrations em container próprio no Docker Compose]]
