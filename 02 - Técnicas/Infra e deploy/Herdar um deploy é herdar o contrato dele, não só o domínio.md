---
tags: [tipo/atomica, camada/padrao, infra, armadilha]
criado: 2026-08-22
---

# Herdar um deploy é herdar o contrato dele, não só o domínio

> Trocar o conteúdo de um repositório que já está no ar parece barato: o domínio, o
> certificado, o DNS e as variáveis continuam de pé, e o push publica. O que não
> continua de pé é o **contrato entre a plataforma e o código** — e ele falha calado.

## Por que é atraente

Reescrita pronta, versão velha no ar. Subir serviço novo significa domínio novo,
certificado novo, DNS novo, migrar variável, e uma janela em que os dois existem.
Empurrar a árvore nova pro repositório antigo pula tudo isso: o deploy que já roda
publica a versão nova sozinha.

O plano está certo. O que se subestima é **quanto da plataforma está descrito dentro
do repositório antigo**, e não no painel.

## O modo de falha

Não é erro de compilação, e é isso que faz doer: o build passa, o deploy roda, e a
plataforma **não promove**. O site velho continua servindo, e o log diz que a
verificação falhou — não que uma rota deixou de existir.

Numa troca real (piwdex v1 → v2, ago/2026) eram três, e duas eram bloqueio:

| o que o repositório antigo declarava | o que quebra |
|---|---|
| `preDeployCommand: node db/setup.mjs` | a pasta `db/` não existe na árvore nova; pré-deploy falha, deploy nunca sobe |
| `healthcheckPath: /api/health` | a árvore nova não tinha **nenhuma** rota de API; healthcheck 404 marca o deploy como falho |
| redirect `www` → apex no `next.config` | não bloqueia nada: o site sobe, responde em dois endereços e vira conteúdo duplicado na busca |

O terceiro é o mais perigoso justamente porque **não** bloqueia. Ninguém percebe até
o tráfego cair.

## O inventário, antes de trocar a árvore

Leia o repositório ANTIGO procurando o que a plataforma consome, não o que o produto
faz:

- **arquivo de configuração da plataforma** (`railway.json`, `vercel.json`,
  `fly.toml`, `Procfile`, `nixpacks.toml`) — cada chave dele é uma promessa: comando
  de pré-deploy, caminho de healthcheck, política de restart, número de réplicas;
- **rotas que só a plataforma chama** — healthcheck, webhook de pagamento, callback
  de OAuth. Não aparecem em nenhum menu e nenhum teste de tela pega;
- **redirect e header no `next.config`/middleware** — canônico, HSTS, `www`;
- **rota pública com link salvo fora do seu controle** — no produto de terceiro, no
  Discord, no favorito de quem usava.

## O que fazer com a rota que morreu

A do cockpit do robô estava salva no próprio jogo e no Discord. Mandar essa gente pro
404 é a pior despedida; a home ao menos mostra o que o site virou.

E o redirect vai **temporário** (307), não permanente. `permanent` ensina navegador e
buscador a nunca mais pedir a rota original, e é caro de desfazer — quem cacheou o 301
não volta. Rota de uma feature *parqueada* não é rota morta: temporário é a afirmação
honesta.

## A reserva vem antes, e é de três jeitos

Tag anotada, branch, e um `git bundle` fora do repositório. Os dois primeiros vivem no
mesmo lugar que você está prestes a mexer; o bundle é um arquivo só, com o histórico
completo, e `git bundle verify` prova que ele reconstrói tudo. Fazer a reserva **antes
de tocar na árvore**, não depois de o push falhar.

## Preserve as duas histórias

Dá pra trocar a árvore inteira mantendo os dois passados no grafo — a commit fica com
dois pais e a árvore fica idêntica à nova:

```bash
git merge -s ours --no-commit --allow-unrelated-histories novo/main
git read-tree -u --reset novo/main     # índice e working tree viram os do novo
git commit
```

Confira antes de empurrar, porque a mão treme:

```bash
[ "$(git rev-parse HEAD^{tree})" = "$(git rev-parse novo/main^{tree})" ] && echo IDENTICA
```

## Conexões
- Princípio: [[Verificar no build de produção, não só em dev]] ·
  [[Migração de dados mantém o antigo como reserva até a virada]]
- Irmã: [[Slot de anúncio no App Router precisa de casca estável e filho keyado]]
- Visto em: [[piwdex2]] · [[piwdex]]
- Mapa: [[Infra]]
