---
tags: [tipo/atomica, camada/padrao, dev/backend]
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

## Duas classes de arquivo pedem duas rotas

Quando o mesmo sistema guarda arquivo que é **produto** (foto do anúncio) e
arquivo que é **prova** (documento de identidade da verificação), a tentação é
uma rota só com um campo de tipo. Não compensa: um `if` errado numa refatoração
futura entrega o RG de alguém pela rota que serve a galeria.

Separe de ponta a ponta — pasta própria no volume, model próprio no banco e rota
própria com autorização própria. A rota pública nem sabe montar o caminho do
documento. E no documento, `Cache-Control: no-store`, não `private`: prova de
identidade não fica em cache de navegador.

A mesma rota pública ainda pode ter uma terceira dimensão além da permissão: o
**paywall**. Mídia marcada como exclusiva de assinante responde 403 para quem não
assina, e a checagem mora ali, não na tela que monta a galeria — a URL do arquivo
é adivinhável.

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
- `Content-Security-Policy: default-src 'none'; sandbox` na resposta do arquivo,
  para que nada do que subiu possa se comportar como página.
- Confira que o arquivo existe antes de abrir o stream:
  [[Linha no banco não garante o arquivo no disco]].

## Conexões
- Princípio: [[Permissão se valida no servidor, não na interface]]
- Ver também: [[Migrations em container próprio no Docker Compose]]
- Irmã: [[Linha no banco não garante o arquivo no disco]]
- Visto em: [[Navedesk]] · [[Privello]]
- Mapa: [[Backend]]
