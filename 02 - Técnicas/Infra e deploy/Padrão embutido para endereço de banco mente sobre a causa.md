---
tags: [tipo/atomica, camada/padrao, infra, armadilha]
criado: 2026-09-02
---

# Padrão embutido para endereço de banco mente sobre a causa

> `process.env.APP_DB_URL ?? "postgres://nexo:nexo@localhost:5022/nexo"` parece
> conveniência para o desenvolvimento. Quando o `.env` não existe, o app conecta
> em ALGUMA COISA com uma senha adivinhada e reporta **"password authentication
> failed"** — e quem lê passa a investigar credencial, que está certa, enquanto o
> que faltava era o arquivo inteiro.

## O problema

O padrão embutido não evita a falha; ele **troca uma falha exata por uma
plausível**. E a plausível é pior, porque manda a investigação para o lugar
errado com uma pista convincente:

| O que faltava | O que o erro disse | Onde a pessoa foi procurar |
|---|---|---|
| o `.env` inteiro | senha do usuário `nexo` recusada | senha, `pg_hba`, volume do Postgres |

Piora quando existe um banco de outra versão do projeto escutando naquela porta:
o endereço adivinhado **acerta um servidor real**, e aí a mensagem fica ainda
mais convincente.

## A solução

Endereço de banco não tem padrão. Ausente, falha dizendo o nome da variável e o
passo que resolve:

```ts
function urlBancoApp(): string {
  const url = process.env.APP_DB_URL?.trim();
  if (!url) {
    throw new ErroBancoApp(
      "APP_DB_URL não está definida — falta o arquivo .env. " +
        "Copie o modelo (`cp .env.example .env`) e preencha APP_DB_PASSWORD e APP_DB_URL."
    );
  }
  return url;
}
```

E o pool nasce **preguiçoso**, no primeiro uso: criado no carregamento do
módulo, o erro estoura no `import` e o rastro aponta para quem importou, não
para a consulta que precisava do banco.

## Onde a linha passa

Nem toda variável merece esse rigor — o critério é **o que o padrão errado
produz**:

- **Ajuste numérico** (tamanho de pool, tempo limite): padrão embutido é certo. O
  valor errado degrada, não mente.
- **Endereço, credencial, segredo**: sem padrão. O valor errado conecta no lugar
  errado, ou falha acusando outra coisa.

O mesmo vale no Compose, e lá a sintaxe já ajuda:
`${APP_DB_PASSWORD:?defina APP_DB_PASSWORD no .env}` recusa subir sem a variável.

## O que mais vale lembrar

- O sintoma de que isso está solto é **a mensagem de erro citar uma coisa que
  ninguém configurou**. Senha recusada de um usuário que você não escolheu, host
  que você não digitou: é padrão embutido falando.
- Vale para os scripts também (migration, seed), e eles são os mais esquecidos —
  rodam fora do app e costumam trazer a mesma string copiada.

## Conexões
- Princípio: [[Configuração vem do ambiente, não do código]]
- Irmã: [[Retry que reusa o cliente queimado esconde o erro da primeira tentativa]] · [[Volume de dev sobrevive entre versões do projeto e traz schema velho]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Infra]]
