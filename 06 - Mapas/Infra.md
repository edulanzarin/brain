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
| [[Questor Hub]] | 4022 | 5022 |
| [[Navecon Controller]] | 4088 | 5432 |
| [[Evento Navecon]] | 4099 | 5099 |

App `4xxx`, banco espelha trocando o `4` inicial por `5`. Regra e exceções em
[[Uma faixa de portas por projeto]].

## Técnicas

`02 - Técnicas/Infra e deploy`

- [[Migrations em container próprio no Docker Compose]] — migration não sobe com o app.
- [[Next.js standalone no Docker e o outputFileTracingRoot]] — imagem enxuta sem
  quebrar o trace de arquivos.

Relacionado, no [[Backend]]:
[[Polling substitui webhook quando não há IP público]] — integração sem abrir porta.

## Princípios que mandam aqui

- [[Configuração vem do ambiente, não do código]] — a raiz de todas as outras.
- [[Permissão se valida no servidor, não na interface]]
- [[Plataforma de IA hospedada prende o app pelo banco]] — cuidado ao escolher onde hospedar.

---

Voltar para [[Início]]
