---
tags: [tipo/atomica, camada/padrao, dev/backend]
criado: 2026-07-19
---

# Armadilhas de child_process no Node

> `execFile` e `execFileSync` têm assinaturas parecidas e opções DIFERENTES. A opção `input` só existe nas versões síncronas — passá-la para a assíncrona não dá erro, o processo simplesmente fica esperando stdin para sempre.

## O bug

```ts
// TRAVA: execFile ignora `input`, o processo espera stdin e nunca termina
const exec = promisify(execFile);
await exec("pdftotext", ["-layout", "-", "-"], { input: bytes });
```

Não estoura exceção, não loga nada — a requisição fica pendurada até o timeout do servidor. Foi o que aconteceu ao mandar PDF por stdin no [[Navetech Hub]].

## O jeito certo

`spawn`, que é o único que dá acesso ao stdin do processo:

```ts
const p = spawn("pdftotext", ["-layout", "-", "-"]);
p.stdout.on("data", (d) => saida.push(d));
p.stderr.on("data", (d) => (erro += d));
p.on("close", (code) => { /* … */ });
p.stdin.on("error", () => {});   // processo pode morrer antes de ler tudo
p.stdin.end(bytes);
```

## O que mais vale lembrar

- **Sempre pôr timeout** com `kill("SIGKILL")`: arquivo corrompido não pode segurar uma conexão HTTP indefinidamente.
- **Capturar stderr**: é lá que vem a causa real ("Incorrect password", "Permission denied"), e sem isso a mensagem ao usuário vira um genérico inútil.
- **`p.stdin.on("error")` vazio** evita `EPIPE` derrubar o processo quando o filho fecha a entrada antes de consumir tudo.
- Preferir `spawn`/`execFile` a `exec`: `exec` passa pelo shell, então argumento com caractere especial vira injeção.

## `-` como entrada depende da implementação (set/2026)

O mesmo nome de programa pode ser duas ferramentas. O `pdftotext` do contêiner é o
**poppler**, que aceita `-` como arquivo de entrada; o do Git for Windows é o
**xpdf**, que não aceita: imprime a tela de ajuda e sai com código 99. Arquivo
temporário (`mkdtemp` + `rm` no `finally`) funciona nas duas e dispensa o stdin.

O defeito apareceu disfarçado por outro: o código reconhecia PDF protegido testando
`/password/i` no stderr — e a tela de ajuda lista `-opw <string> : owner password`.
Todo PDF, protegido ou não, virava pedido de senha. **Classificar a falha de
ferramenta externa pela frase exata** que ela emite naquele caso (`Incorrect
password`, igual nas duas), nunca pela palavra solta, que aparece em ajuda, aviso
e mensagem de outro erro.

## Conexões
- Princípio: [[Chamada externa tem timeout e erro tratado]]
- Visto em: [[Ler extrato bancário em PDF]] · [[Navetech Hub]]
- Mapa: [[Backend]]
