---
tags: [tipo/atomica, camada/principio, infra]
criado: 2026-07-20
---

# Uma faixa de portas por projeto

> Cada projeto tem sua faixa reservada. Escolher porta deixa de ser adivinhação e
> vira consulta a uma lista.

## A regra

O app fica na faixa `4xxx` (4001–4999) — a faixa das automações. **O banco espelha o
app trocando o `4` inicial por `5`**: app `4004` → banco `5004`, app `4011` → banco
`5011`. Os três últimos dígitos são os mesmos, então a porta identifica o projeto
sozinha e eu nunca preciso testar se está livre.

Mapa atual:

| Projeto | App | Banco |
|---|---|---|
| [[Cofre Digital]] | 4004 | 5004 |
| [[Navedesk]] | 4001 | 5401 |
| [[Navetech Hub]] | 4022 | 5022 |
| [[Navecon Controller]] | 4088 | 5432 |
| [[Evento Navecon]] | 4099 | 5099 |
| [[navetalks]] | 4050 | 5050 |
| [[Navehub]] | 4030 | 5030 |
| [[Navecon CRM]] | 4046 | 5046 |
| [[Idle Game]] | 4060 | 5060 |
| [[piwdex]] | 4070 | 5070 |
| [[Vespéria]] | 4073 | 5073 |
| [[piwdex2]] | 4071 · 4072 | 5071 |
| [[monofire]] | 4074 | 5074 |
| [[Privello]] | 4075 | 5075 |
| privello2 | 4076 | 5076 |
| [[CRM Contábil]] | 4077 | 5077 |
| Central Contábil | 4010 | 5010 |

Projeto novo pega o próximo número livre e **registra aqui na hora** — a nota é a fonte
da verdade, não o `docker ps` da máquina que por acaso está ligada.

## Um projeto pode ter duas portas de app, e um banco só

O par nasceu supondo um processo web por projeto, e o [[piwdex2]] quebrou a
suposição em ago/2026: ele publica **dois** endereços (a dex e o robô) a partir
da mesma imagem, porque as duas metades precisam de cadências de deploy
diferentes — ver
[[Um processo serve dois hosts quando o papel vem do ambiente]].

A regra continua a mesma, só que o `4xxx` identifica **o processo**, não o
projeto: piwdex2 fica com 4071 (dex) e 4072 (robô), e um banco só, o 5071. O
espelho `4`→`5` vale pro par principal; a segunda porta de app não ganha banco
espelhado porque não há um segundo banco.

Reservar 4072 tira esse número da fila de projetos novos. É o preço, e é barato:
a faixa tem 999 números.

## O terceiro serviço fica em 6xxx, com os mesmos três dígitos

Quando o projeto precisa publicar um processo que não é o app nem o banco — um
agendador, um worker, uma fila, um cache — ele vai pra faixa `6xxx` mantendo os três
últimos dígitos: app `4010`, banco `5010`, terceiro `6010`. O número continua
identificando o projeto sozinho, agora em três faixas em vez de duas.

**A faixa é o papel, não a ordem de criação.** Quem o navegador alcança fica em `4xxx`,
mesmo sendo o segundo processo web — é por isso que o [[piwdex2]] tem 4071 e 4072, e não
4071 e 6071. Banco fica em `5xxx`. O `6xxx` é pra quem não é nenhum dos dois.

**Antes de reservar, confirme que o terceiro serviço precisa mesmo de porta.** A maioria
não precisa: um agendador sobe como serviço do compose sem publicar nada e fala com o
banco pela rede interna — ver
[[Agenda recorrente é um serviço do compose, não um crontab do host]]. Porta publicada é
superfície exposta; reservar pra quem não vai usar é abrir buraco à toa. Hoje nenhum
projeto ocupa a faixa `6xxx`, e isso é o esperado.

No `6xxx` as portas recusadas pelo navegador são a 6000 (X11) e a 6665–6669 e 6697
(IRC). Só mordem se o serviço for aberto no navegador, mas pule esses números mesmo
assim — o espelho só serve se valer nas três faixas ao mesmo tempo.

## Uma porta livre ainda pode ser proibida pelo navegador

A porta escolhida não pode ser só "livre no host" — tem que ser aceita pelo **navegador**.
Chromium e Firefox mantêm uma lista de portas *unsafe* (herança de serviços antigos:
lockd, X11, IRC…) e recusam conectar nelas; o Next inclusive se recusa a subir com
`next start -p 4045` ("Bad port: 4045 is reserved for npp"). Por isso Navecon CRM ficou em
**4046**, não 4045 (o próximo número na sequência caía justo numa porta bloqueada).

A regra do espelho `4xxx`→`5xxx` continua, mas ao reservar o par, pule qualquer porta da
lista unsafe do navegador — no range `4xxx` a que morde é a **4045**. Visto em
[[Navecon CRM]] (ago/2026).

## A tabela só é fonte da verdade se o projeto novo se registrar nela

A regra diz para consultar aqui e não o `docker ps`, e ela continua certa — mas ela
supõe que quem cria projeto **escreve nesta tabela na hora**. Em set/2026 o par
4076/5076 estava livre segundo a nota e ocupado de fato pelo privello2, que nunca foi
registrado. O conflito só apareceu no `docker compose up`, depois de o slug, o `.env`,
o compose e o `package.json` já estarem escritos com o número errado.

O custo foi baixo (um `sed`), e é baixo justamente enquanto o projeto é novo. Duas
consequências práticas:

- **Reservar é escrever aqui, não escolher mentalmente.** Projeto que ainda não tem
  nota entra na tabela mesmo assim, pelo nome da pasta.
- **Confira o par contra a máquina antes de cravar**, mesmo confiando na tabela. Um
  `docker ps` custa segundos e cobre o caso de alguém — inclusive você — ter pulado o
  registro. A tabela manda; a máquina desempata quando as duas discordam.

## O par são duas portas, não uma

"O projeto roda na 4004" quer dizer que **o app** atende em 4004. O banco do mesmo
projeto não atende na 4004 também — duas coisas não escutam a mesma porta do mesmo
host. Daí o par: `4004` app, `5004` banco, o mesmo número com o primeiro dígito trocado.

E o `5xxx` não briga com o Postgres "padrão": 5432 é a porta **dentro** do container, e
segue 5432 em todo projeto. O `5004` é só onde o host publica aquele container — ver
[[Porta interna é constante, porta externa é configuração]].

## Por que

O erro que isso evita não é o conflito de porta (esse o Docker grita na hora). É o
silencioso: dois projetos com o mesmo banco em 5432, eu abro o cliente SQL apontando
pro que achei que era, e rodo query no banco errado. Em ago/2026 isso quase aconteceu no
Central Contábil, com um PostgreSQL instalado nativo no Windows disputando a 5432 com o
container — ver
[[No Windows, duas coisas escutam a mesma porta e o cliente fala com a errada]]. Com faixa reservada, a porta
identifica o projeto sozinha.

Cofre Digital e Navetech Hub seguem o espelho `4xxx`→`5xxx`. Os que vieram antes e ainda
não seguem: Navecon Controller (5432) e Navedesk (5401, formato antigo `54xx`) — ficam
como estão, repontar porta de projeto rodando não paga o risco; o espelho vale daqui pra
frente. O Navetech Hub era exceção (5433) e passou pro espelho (5022) na renomeação de
jul/2026, aproveitando que já estava mexendo em tudo — a hora barata de repontar é
quando o projeto já vai mudar de nome mesmo.

## Conexões
- Irmã: [[Porta interna é constante, porta externa é configuração]] · [[O nome do projeto governa o nome dos recursos]]
- Mapa: [[Base]] · [[Infra]]
