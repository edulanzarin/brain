---
tags: [tipo/atomica, camada/padrao, infra, dev/frontend, armadilha]
criado: 2026-09-05
---

# Variável pública é embutida no build; endereço de produção é de execução

> `NEXT_PUBLIC_SITE_URL` no compose, passada como ambiente do container, com o
> Dockerfile construindo numa etapa que não a tem. Parece configuração; é
> constante. O empacotador já trocou a expressão pelo literal, e o que ficou
> gravado na imagem foi o padrão de desenvolvimento.

## O mecanismo

O prefixo que marca a variável como pública (`NEXT_PUBLIC_` no Next, `VITE_` no
Vite, `PUBLIC_` no SvelteKit) não significa "pode ser lida por qualquer um em
tempo de execução". Significa **"pode ir para o navegador"** — e a única forma
de mandar algo para o navegador é escrever no pacote. Então o empacotador
substitui `process.env.NEXT_PUBLIC_X` pelo valor literal enquanto constrói.

Depois disso não existe mais leitura de ambiente naquele ponto do código. Passar
a variável ao container não muda nada: não há o que ler.

O que torna a armadilha cara é onde ela aparece. Num Dockerfile multi-etapa, o
build roda numa etapa e o `environment:` do compose vale na outra:

```dockerfile
FROM base AS builder
COPY . .
RUN npm run build          # aqui a variável não existe → grava o padrão
...
FROM base AS runner
CMD ["node", "server.js"]  # aqui ela existe, e é tarde
```

## O sintoma

Nenhum. Os tipos passam, o lint passa, o build passa, o container sobe e o site
funciona. Só que **canônico, sitemap, dado estruturado e cartão de
compartilhamento saem todos apontando para `localhost`** — o que, num site cujo
tráfego inteiro vem de busca, é o site se apagando do índice em silêncio.

## A forma

Se o valor só é usado no servidor, **tire o prefixo**. Sem ele a variável é lida
do ambiente a cada requisição, e a mesma imagem sobe em qualquer domínio — que é
o que já se faz com porta e com endereço de banco.

```ts
import "server-only";
export const SITE = process.env.SITE_URL ?? "https://exemplo.com.br";
```

O `import "server-only"` não é enfeite: é o que faz o build QUEBRAR se alguém
importar isso de um componente de cliente, que é exatamente o caminho de volta
para o problema. Junto com ele, tirar o módulo do barril de componentes — barril
é importado por componente de cliente, e basta o encosto para arrastar a cadeia.

Se o valor for mesmo necessário no navegador, aí ele é constante de build por
natureza: passe como `ARG` na etapa que constrói, e aceite que trocar de domínio
exige reconstruir a imagem.

## O padrão aponta para produção, não para dev

As duas formas de errar não custam igual. Um canônico apontando para o domínio
real numa máquina de desenvolvimento não indexa nada e ninguém vê; o inverso
apaga o site do índice. O padrão embutido deve ser o caso caro.

## Conexões
- Princípio: [[Configuração vem do ambiente, não do código]]
- Irmã: [[Padrão embutido para endereço de banco mente sobre a causa]] — o mesmo
  vício, com o outro sintoma: lá o padrão esconde a causa do erro, aqui ele
  substitui a verdade sem erro nenhum.
- Irmã: [[Metadata do Next não funde o aninhado, e o que herda vaza]]
- Visto em: [[Privello]]
- Mapa: [[Infra]] · [[Frontend]]
