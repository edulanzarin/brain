---
tags: [tipo/atomica, camada/padrao, infra, dev/frontend, armadilha]
criado: 2026-09-17
---

# No Windows o npm roda script pelo cmd.exe, e a porta padrão do script dev chega literal

> `"dev": "next dev -p ${PORT:-4022}"` é sintaxe de shell POSIX. No Windows o npm
> executa scripts pelo `cmd.exe`, que não expande nada disso, e o Next recebe o texto
> cru como porta.

## O problema

A convenção de porta ([[Porta interna é constante, porta externa é configuração]])
põe o padrão dentro do próprio script: `${PORT:-40xx}`. No Linux e no macOS o npm roda
o script em `sh`, que troca a expressão por `4022` ou pelo `PORT` do ambiente. No
Windows o shell padrão do npm é `cmd.exe /d /s /c`, que não conhece `${...}` e passa a
string adiante do jeito que está.

Dá para provar sem subir servidor nenhum, com um script que só imprime o argumento:

```json
{ "scripts": { "t": "node -e \"console.log(process.argv[1])\" ${PORT:-4022}" } }
```

`npm run t` no Windows imprime `${PORT:-4022}`. É isso que o `next dev -p` recebe.

## A solução

Apontar o `script-shell` do npm para o Bash do Git, uma vez por máquina:

```bash
npm config set script-shell "C:\Program Files\Git\bin\bash.exe"
```

Fica no `~/.npmrc` do usuário. O mesmo teste passa a imprimir `4022`, e
`PORT=4999 npm run t` imprime `4999`. Desfazer é `npm config delete script-shell`.

Por que na máquina e não no projeto: os projetos foram escritos para shell POSIX (os
scripts, o entrypoint, a própria convenção), então o shell certo para rodá-los é o
POSIX. Trocar a convenção por uma versão que funcione no `cmd` (pacote extra só para
ler variável, ou padrão de porta dentro do `next.config`) mexeria em todo repositório
por causa de uma máquina.

## O que mais vale lembrar

- `npm run build`, `npm test` e scripts sem `$` funcionam no `cmd`. O defeito só
  aparece no script que depende de expansão, e a mensagem do Next ("porta inválida")
  não diz que a culpa é do shell.
- `PORT=4999 npm run dev` com a variável na frente também é sintaxe de Bash. No
  PowerShell é `$env:PORT=4999; npm run dev`, e com o `script-shell` apontado para o
  Bash o `${PORT:-...}` enxerga essa variável normalmente.
- Um `.npmrc` versionado no repositório com esse caminho não serve: o caminho do Git
  muda de máquina para máquina e quebraria o `npm` no Linux.

## Conexões
- Princípio: [[Porta interna é constante, porta externa é configuração]]
- Irmã: [[Sem virtualização na BIOS não há Docker no Windows; o banco de dev vira Postgres portátil]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Infra]]
