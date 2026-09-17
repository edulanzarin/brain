---
tags: [tipo/atomica, camada/padrao, infra, docker, armadilha]
criado: 2026-09-17
---

# Sem virtualização na BIOS não há Docker no Windows; o banco de dev vira Postgres portátil

> Docker Desktop precisa de WSL2 ou Hyper-V, e os dois precisam da virtualização
> ligada no firmware. Sem ela, o banco de dev é um Postgres portátil na porta que o
> projeto já reservou, e o `.env` não muda nada.

## O problema

A convenção de projeto sobe o banco de dev em container (`npm run db:up`). Num
Windows 11 recém-formatado, sem Docker, a primeira ideia é `winget install
Docker.DockerDesktop`. Não adianta nada se a máquina não virtualiza. Dá para conferir
antes de instalar:

```powershell
(Get-CimInstance Win32_Processor).VirtualizationFirmwareEnabled   # False = não roda
```

Com `False`, o WSL2 e o Hyper-V não sobem, e o Docker Desktop instala sem conseguir
iniciar. Ligar VT-x/AMD-V é reiniciar e entrar na BIOS, uma decisão do dono da máquina
e não um passo de script.

## A solução

Postgres portátil: binários numa pasta do usuário e um cluster por projeto. Não
precisa de admin, não vira serviço do Windows e sai apagando duas pastas.

1. **Binários.** O pacote npm `@embedded-postgres/windows-x64` traz o Postgres
   oficial já compilado, com versão igual à do projeto (17.x). É só extrair, sem
   instalador:

   ```bash
   npm pack @embedded-postgres/windows-x64@17.10.0-beta.17
   tar -xzf embedded-postgres-windows-x64-*.tgz
   cp -r package/native "$LOCALAPPDATA/Programs/pgsql-17"
   ```

2. **Runtime do Visual C++.** Se o `postgres.exe` morre com código `-1073741515`
   (`0xC0000135`, DLL não encontrada) e o Git Bash acusa `api-ms-win-crt-*.dll`,
   falta o `vcruntime140.dll`. Uma instalação limpa do Windows não traz:
   `winget install Microsoft.VCRedist.2015+.x64` (esse pede admin uma vez).

3. **Cluster por projeto**, com usuário e senha do `.env`:

   ```bash
   initdb -D "$LOCALAPPDATA/pgdata/<slug>" -U <slug> --pwfile=<arquivo-temporario> \
     --auth=scram-sha-256 -E UTF8 --locale=C
   ```

   E no `postgresql.conf`:

   ```
   port = 5xxx                  # a porta do projeto em docker-compose.dev.yml
   listen_addresses = 'localhost'
   ```

4. Subir com `pg_ctl -D ... -l ...\<slug>.log -w start`, criar o banco e rodar as
   migrations do projeto normalmente.

## O pacote traz o servidor, não o cliente

`@embedded-postgres/windows-x64` entrega **três** executáveis: `initdb.exe`,
`pg_ctl.exe` e `postgres.exe`. As ferramentas de cliente não vêm junto — não há
`psql`, `createdb` nem `pg_dump` nessa pasta, e quem for procurá-las lá vai achar
que a instalação quebrou.

Isso morde no passo 4, porque `initdb` cria o cluster mas **não** cria o banco do
projeto: sem `createdb`, a conexão falha em `InitPostgres` com uma mensagem que
não diz que o banco não existe.

A saída não é instalar o Postgres completo, é usar o cliente que o projeto já
tem. O `pg` do Node conecta no banco `postgres` (esse o `initdb` sempre cria) e
roda o `create database`:

```js
const c = new pg.Client({ host: "localhost", port: PORTA, user: SLUG,
                          password: SENHA, database: "postgres" });
await c.connect();
const { rows } = await c.query("select 1 from pg_database where datname = $1", [SLUG]);
if (!rows.length) await c.query(`create database ${SLUG}`);
```

Pelo mesmo motivo, o `db:psql` da convenção não existe aqui. No lugar dele vale
um `db:sql "select ..."` que executa pelo `pg` e imprime com `console.table` —
resolve a consulta rápida sem binário nenhum a mais.

## `pg_ctl start` com stdio herdado prende quem chamou

`spawnSync(pg_ctl, ["start"], { stdio: "inherit" })` retorna, mas o terminal
**não volta**: o `postgres` herda os descritores do `pg_ctl` e os mantém abertos
enquanto viver, então quem chamou o `npm run db:up` fica pendurado esperando um
cano que só fecha quando o banco cair.

O sintoma engana: o banco sobe, o cluster funciona, e parece que o script travou.
`stdio: "ignore"` resolve — o log já vai para arquivo pelo `-l`, que é onde ele
deve estar. Caso particular de [[Armadilhas de child_process no Node]].

E, no fim do script, `rmSync` da pasta temporária pode estourar `EBUSY` no
Windows, porque apagar arquivo ainda aberto não é permitido; envolva em
`try/catch` em vez de deixar a limpeza derrubar um trabalho já concluído.

## O que mais vale lembrar

- **A porta é a reservada do projeto**, não 5432. O `APP_DB_URL` de dev já aponta
  para `localhost:5xxx`, então o app não sabe se do outro lado tem container ou
  processo nativo. E a 5432 é justamente a que algum Postgres alheio já ocupa (ver
  [[No Windows, duas coisas escutam a mesma porta e o cliente fala com a errada]]).
- **`listen_addresses = 'localhost'`, não `127.0.0.1`.** O Node resolve `localhost`
  primeiro para `::1`; escutando só IPv4, a conexão é recusada com a URL certa.
- **A collation segue a imagem de produção, não o gosto.** `postgres:*-alpine` roda
  sobre musl, que não implementa ordenação por idioma: lá o texto ordena por byte
  (`Zé < ana < Álvaro`). `--locale=C` reproduz isso. Um cluster ICU `en-US` ordenaria
  bonito em dev e esconderia a lista que sai torta em produção, que é exatamente a
  divergência que [[Ambiente de dev sobe igual ao de produção]] manda evitar. Se a
  imagem for Debian (`postgres:17` sem `-alpine`), aí sim o equivalente é
  `en_US.UTF-8`.
- **É exceção, não troca de convenção.** Produção continua em compose. O que fica
  fora do container em dev (o `pdftotext` que a imagem instala, por exemplo) vira
  diferença a lembrar, e o build de produção continua sendo onde se verifica.
- Não sobe sozinho no boot. Depois de reiniciar, é `pg_ctl ... start` de novo.

## Conexões
- Princípio: [[Ambiente de dev sobe igual ao de produção]] · [[Uma faixa de portas por projeto]]
- Irmã: [[No Windows o npm roda script pelo cmd.exe, e a porta padrão do script dev chega literal]] · [[No Windows, duas coisas escutam a mesma porta e o cliente fala com a errada]] · [[Formatar a máquina perde tudo que o git não versiona]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Infra]]
