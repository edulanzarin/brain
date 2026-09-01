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
   [[Uma faixa de portas por projeto]]. Terceiro serviço com porta, se houver, em `6xxx`.

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
| [[Vespéria]] | 4073 | 5073 |
| [[piwdex2]] | 4071 | 5071 |
| [[monofire]] | 4074 | 5074 |
| [[Privello]] | 4075 | 5075 |
| privello2 | 4076 | 5076 |
| CRM Contábil (arquivado) | 4077 | 5077 |
| [[navecrm]] | 4078 | 5078 |
| Central Contábil | 4010 | 5010 |

App `4xxx`, banco espelha trocando o `4` inicial por `5`, e um terceiro serviço que
precise de porta (agendador, worker, fila) vai pra `6xxx` com os mesmos três dígitos:
4010 · 5010 · 6010. A faixa é o papel, não a ordem — segundo processo web continua em
`4xxx`. Regra e exceções (incluindo as portas *unsafe* que o navegador recusa, como a
4045) em [[Uma faixa de portas por projeto]].

## Técnicas

`02 - Técnicas/Infra e deploy`

- [[Migrations em container próprio no Docker Compose]] — migration não sobe com o app.
- [[Runner de migration em SQL puro dispensa o CLI do ORM]] — sem ORM, o migrator é
  um script de ~30 linhas; some o peso do CLI do Prisma na imagem standalone.
- [[Next.js standalone no Docker e o outputFileTracingRoot]] — imagem enxuta sem
  quebrar o trace de arquivos.
- [[App sob subcaminho fica na raiz e o proxy tira o prefixo]] — servir em
  `/imersao` sem remontar o app; base única no front, strip no proxy.
- [[No Windows, duas coisas escutam a mesma porta e o cliente fala com a errada]] — o
  Windows aceita dois LISTENING na mesma porta e o cliente fala com o errado; o idioma
  da mensagem de erro denuncia qual servidor respondeu, e cada erro diz até que camada
  a conexão chegou.
- [[Volume de dev sobrevive entre versões do projeto e traz schema velho]] — rebuild
  no mesmo slug reencontra o banco antigo; recriar o volume, não forçar reset.
- [[Renomear coluna é migration à mão; a gerada derruba e recria]] — o ORM compara
  schemas e não lê intenção: coluna que troca de nome vira uma que sumiu e outra que
  nasceu, e o conteúdo vai junto. `RENAME` no SQL escrito à mão, com o índice único
  renomeado junto e a transformação de conteúdo dentro da mesma migration.
- [[Nome de migration do Prisma é UTC, e é o nome que ordena]] — pasta carimbada com
  a hora local ordena antes das que já existem. Aplica sem reclamar no banco de dev e
  quebra na primeira subida de um ambiente limpo, que é o que a produção faz.
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
- [[Retry que reusa o cliente queimado esconde o erro da primeira tentativa]] — laço de
  espera pelo banco que reusa o mesmo `pg.Client` reporta o próprio defeito, e a causa
  real (senha errada) some na primeira volta.
- [[Processo que guarda conexão viva não tolera deploy frequente, e o log não denuncia]] —
  serviço com WebSocket/sessão em memória morre inteiro a cada deploy, e com auto-deploy
  cada push derruba todo mundo. O log escreve "Ready" igual nos dois casos; quem denuncia
  é uma sonda de `uptimeSeconds`. Compare a vida do processo com a menor janela interna
  que ele precisa cumprir.
- [[Um processo serve dois hosts quando o papel vem do ambiente]] — o remédio da nota
  acima: cadência de deploy é propriedade do SERVIÇO, não do repositório. A mesma imagem
  sobe duas vezes e uma variável decide qual host cada processo atende, sem duplicar o
  sistema de design. Roteie por lista explícita de rotas, nunca por prefixo — prefixo
  arrasta favicon e imagem social pra um 404 silencioso.
- [[Página que consulta o banco não pode nascer no build]] — o build roda dentro
  do Docker, sem banco, e derruba qualquer rota que o Next resolva pré-renderizar
  com uma query dentro. `revalidate` não salva; `force-dynamic` sim, e ainda
  impede o conteúdo de congelar na data do deploy.
- [[Ponte pro endereço novo só se levanta quando o outro lado responde]] — redirect é
  código, mas o destino é fato de infraestrutura. A variável de ambiente é a declaração
  de que o outro lado existe; sem ela, o comportamento anterior continua valendo.

Relacionado, no [[Backend]]:
[[Polling substitui webhook quando não há IP público]] — integração sem abrir porta.

## Princípios que mandam aqui

- [[Configuração vem do ambiente, não do código]] — a raiz de todas as outras.
- [[Permissão se valida no servidor, não na interface]]
- [[Plataforma de IA hospedada prende o app pelo banco]] — cuidado ao escolher onde hospedar.

---

Voltar para [[Início]]
