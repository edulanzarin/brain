---
tags: [tipo/moc]
criado: 2026-07-20
---

# Infra

Como um projeto sobe: container, porta, deploy, migration. Os princípios estão em
[[Base]]; aqui ficam as receitas concretas em Docker, Compose e Next.

## O chassi de todo projeto novo

Quatro decisões que já têm resposta padrão — não reinventar a cada projeto:

1. **Slug em kebab-case** governa o nome de tudo:
   [[O nome do projeto governa o nome dos recursos]].

   | Recurso | Nome |
   |---|---|
   | App | `<slug>-app` |
   | Banco | `<slug>-db` |
   | Migrations | `<slug>-migrate` |
   | Rede | `<slug>-net` |
   | Volume | `<slug>-db-data` |

2. **Par de portas reservado** — próximo livre, registrado em
   [[Uma faixa de portas por projeto]].

3. **Porta interna fixa** (3000 app, 5432 banco), externa por variável:
   [[Porta interna é constante, porta externa é configuração]].

   ```yaml
   ports: ["${APP_PORT:-4011}:3000"]
   ```

   No `package.json`, um script só: `"dev": "next dev -p ${PORT:-4011}"`.
   Rodar em outra porta é `PORT=3000 npm run dev` — nunca criar `dev:3000`.

4. **Compose igual em dev e produção**, com app, db e migrations:
   [[Ambiente de dev sobe igual ao de produção]].

Seguindo isso, o esqueleto sai igual em todo projeto e os comandos de manutenção viram
template: `docker logs <slug>-app`, `docker exec -it <slug>-db psql`.

## Mapa de portas

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
| [[piwdex]] | 4070 | 5070 |
| [[piwdex2]] | 4071 | 5071 |

App `4xxx`, banco espelha trocando o `4` inicial por `5`. Regra e exceções (incluindo as
portas *unsafe* que o navegador recusa, como a 4045) em [[Uma faixa de portas por projeto]].

## Técnicas

`02 - Técnicas/Infra e deploy`

- [[Migrations em container próprio no Docker Compose]] — migration não sobe com o app.
- [[Runner de migration em SQL puro dispensa o CLI do ORM]] — sem ORM, o migrator é
  um script de ~30 linhas; some o peso do CLI do Prisma na imagem standalone.
- [[Next.js standalone no Docker e o outputFileTracingRoot]] — imagem enxuta sem
  quebrar o trace de arquivos.
- [[App sob subcaminho fica na raiz e o proxy tira o prefixo]] — servir em
  `/imersao` sem remontar o app; base única no front, strip no proxy.
- [[Volume de dev sobrevive entre versões do projeto e traz schema velho]] — rebuild
  no mesmo slug reencontra o banco antigo; recriar o volume, não forçar reset.
- [[Agenda recorrente é um serviço do compose, não um crontab do host]] — o
  agendador dos jobs sobe junto no deploy, não é config manual do servidor.
- [[Railway não roda compose, cada serviço vira uma peça da plataforma]] — o mapa
  compose → Railway: migration vira pre-deploy na própria imagem (que precisa
  carregar `db/`), worker vira loop no processo, banco vira plugin; 1 réplica
  quando há estado singleton.
- [[Herdar um deploy é herdar o contrato dele, não só o domínio]] — trocar a árvore
  de um repositório que já está no ar herda domínio, certificado e variáveis, e
  também o que a plataforma consome de dentro do repo: pre-deploy, healthcheck,
  redirect canônico e rota com link salvo fora do seu controle. Duas dessas falham
  caladas — o deploy não promove e o site velho continua servindo.

Relacionado, no [[Backend]]:
[[Polling substitui webhook quando não há IP público]] — integração sem abrir porta.
- [[Processo que guarda conexão viva não tolera deploy frequente, e o log não denuncia]] —
  serviço com WebSocket/sessão em memória morre inteiro a cada deploy, e com auto-deploy
  cada push derruba todo mundo. O log escreve "Ready" igual nos dois casos; quem denuncia
  é uma sonda de `uptimeSeconds`. Compare a vida do processo com a menor janela interna
  que ele precisa cumprir.

## Princípios que mandam aqui

- [[Configuração vem do ambiente, não do código]] — a raiz de todas as outras.
- [[Permissão se valida no servidor, não na interface]]
- [[Plataforma de IA hospedada prende o app pelo banco]] — cuidado ao escolher onde hospedar.

---

Voltar para [[Início]]
