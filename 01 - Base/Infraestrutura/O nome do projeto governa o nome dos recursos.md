---
tags: [tipo/atomica, camada/principio, infra, docker]
criado: 2026-07-20
---

# O nome do projeto governa o nome dos recursos

> Existe um slug por projeto, e todo recurso que sobe deriva dele por sufixo.
> Nada de nome criativo, nada de nome que só faz sentido no dia em que foi criado.

## A regra

Slug do projeto em kebab-case (`prospects`, `cofre-digital`, `questor-bi`). A partir
dele, sem exceção:

| Recurso | Nome |
|---|---|
| Aplicação | `prospects-app` |
| Banco | `prospects-db` |
| Migrations | `prospects-migrate` |
| Rede | `prospects-net` |
| Volume de dados | `prospects-db-data` |
| Projeto compose | `prospects` |

O nome do compose vem de `COMPOSE_PROJECT_NAME=prospects` no `.env`, não do nome da
pasta — pasta renomeada não pode virar stack órfã. Cada serviço leva `container_name`
explícito, senão o Docker inventa `prospects-app-1` e o sufixo numérico entra em todo
comando que eu digitar depois.

## Por que

Com dez projetos no mesmo host, `docker ps` é a tela mais usada do dia. Se os nomes
seguem a regra, eu leio a coluna de nome e sei na hora **de quem é** cada container e
**o que ele faz** — e `docker ps | grep prospects` me dá a stack inteira do projeto.
Sem regra, viram `db`, `postgres`, `web`, `app-1`, e a única forma de saber de quem é
aquele Postgres é abrir o compose.

O ganho real não é estética, é **operação sob pressão**: derrubar o container certo às
23h é um problema de nomenclatura, não de Docker.

## Consequência

Como o nome é derivável do slug, todo comando de manutenção vira template
(`docker logs prospects-app`, `docker exec -it prospects-db psql`). Não preciso lembrar
de nada específico do projeto — só do slug, que é o nome da pasta.

## Conexões
- Depende de: [[Configuração vem do ambiente, não do código]]
- Padrão que aplica: [[Migrations em container próprio no Docker Compose]]
- Irmã: [[Porta interna é constante, porta externa é configuração]]
- Mapa: [[Base]] · [[Infra]]
