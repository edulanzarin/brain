---
tags: [tipo/atomica, camada/padrao, infra, docker, armadilha]
criado: 2026-08-28
---

# No Windows, duas coisas escutam a mesma porta e o cliente fala com a errada

> O Windows deixa dois processos escutarem a mesma porta sem reclamar. O container
> sobe, o `docker ps` mostra o mapeamento certo, e o cliente conversa com o outro.

## O que aconteceu

Central Contábil rodando em Docker Desktop num Windows da rede (ago/2026). O app
respondia em `IP:4010`, mas o DBeaver não conectava no banco em `IP:5432`. A
sequência de erros, e o que cada um significou:

1. **Porta fechada de fora.** Regra de entrada criada no firewall do Windows
   (`New-NetFirewallRule ... -LocalPort 5432 -Profile Any`) e o TCP abriu.
2. **`FATAL: nenhuma entrada em pg_hba.conf para o hospedeiro "10.81.128.2"`.**
   Só que o `pg_hba.conf` do container terminava em `host all all all
   scram-sha-256` — libera qualquer IP. Se fosse ele, o erro seria de senha.
   Logo, quem recusou era outro Postgres.
3. **`netstat -ano | findstr :5432`** mostrou **dois** LISTENING em `0.0.0.0:5432`,
   com PIDs diferentes, e `Get-Service *postgres*` revelou um
   `postgresql-x64-18` instalado nativo na máquina.

No Linux o segundo bind falharia na hora ("address already in use"). No Windows,
quando o primeiro socket não pede uso exclusivo, o segundo entra em silêncio — e
quem atende a conexão não é necessariamente quem você acha.

## O idioma do erro identificou o servidor antes do netstat

A mensagem veio em português. A imagem `postgres:16-alpine` responde em inglês; o
instalador nativo do PostgreSQL para Windows instala mensagens localizadas. O
idioma do erro já dizia que o servidor do outro lado não era o container — foi o
que apontou pra causa antes de qualquer comando.

## A escada de diagnóstico

Cada erro diz **até que camada** a conexão chegou. Subir na ordem evita mexer na
camada errada:

| O que volta | Parou em |
|---|---|
| timeout / connection refused | rede — porta não publicada, ou firewall |
| `nenhuma entrada em pg_hba.conf` | chegou no Postgres; é ele que recusa a origem |
| autenticação falhou | `pg_hba` passou; o problema é credencial |

O erro de `pg_hba` é bom sinal disfarçado de fracasso: significa que rede e
firewall já estão resolvidos.

## O conserto

Não desligar o Postgres nativo — outra coisa na máquina usa ele. Tirar o
**container** da porta disputada, publicando na porta reservada do projeto:

```yaml
ports:
  - "${DB_PORT:-5010}:5432"
```

E remover a regra de firewall criada pra 5432: ela não abriu o container coisa
nenhuma, abriu o **Postgres nativo** pra rede inteira. Regra de firewall
escrita durante diagnóstico tem que ser revista quando a causa muda.

## A variante que não precisa de duas portas: IPv4 e IPv6 na mesma

Em set/2026, no simulador tributário, o container subiu certo e `curl
localhost:4082/api/health` devolveu **500**, enquanto as páginas devolviam 200.
`Get-NetTCPConnection -LocalPort 4082` explicou:

```
::1          pid=3956  node                  <- vite preview esquecido
127.0.0.1    pid=7656  com.docker.backend    <- o container
```

Dois donos da mesma porta, cada um numa família de endereço, e **nenhum bind
falhou**. No Windows o `localhost` resolve **IPv6 primeiro**, então todo `curl
localhost` foi parar no processo velho; `curl 127.0.0.1` ia no container. O
`/api/health` dava 500 porque o vite tentava fazer proxy pra um backend que não
existia mais.

Duas lições operacionais:

- **Matar a tarefa de background não garante matar o processo.** O `npm`/`npx`
  morre e o `node` filho fica escutando. Confira a porta, não a tarefa.
- **Em diagnóstico, prefira `127.0.0.1` a `localhost`.** Tira a resolução de
  nome da equação. Se o comportamento muda entre os dois, a causa é esta.

## O sintoma que denuncia "não é o meu servidor"

Antes de qualquer comando de rede, dois sinais bastaram:

- **Nada no log do container.** Uma requisição que dá 500 no meu Express passa
  pelo handler de erro e imprime `[error]`. Log limpo com erro na tela significa
  que a requisição não chegou ali.
- **Header que não é meu.** A resposta vinha `Content-Type: text/plain` com
  `Vary: Origin` e **sem** o `Content-Security-Policy` que o helmet põe em toda
  resposta minha. Cabeçalho ausente é assinatura: se o middleware obrigatório
  não carimbou, quem respondeu foi outro.

Vale como regra geral, fora do Windows: **compare a resposta com a assinatura do
seu próprio stack antes de debugar a rota.**

## Por que a faixa reservada resolve isso na origem

Publicar banco na 5432 é a única escolha que **garante** encontro com um Postgres
alheio, porque 5432 é o padrão de todo mundo — instalador nativo, colega, outro
projeto. [[Uma faixa de portas por projeto]] existe pra evitar exatamente o erro
silencioso de rodar query no banco errado; aqui ele apareceu na forma barulhenta,
que é a sorte grande. A versão silenciosa teria conectado.

## Conexões
- Princípio: [[Uma faixa de portas por projeto]]
- Irmã: [[Porta interna é constante, porta externa é configuração]] ·
  [[Config declarada envelhece; quem diz a regra é o comportamento observado]]
- Visto em: [[Simulador Navecon]]
- Mapa: [[Infra]]
