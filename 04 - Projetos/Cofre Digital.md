---
tags: [tipo/projeto, projeto/cofre-digital]
criado: 2026-07-20
status: ativo
codigo_em: ~/Dev/cofre-digital
---

# Cofre Digital

> Cofre da intranet Navecon: certificados digitais (e-CNPJ, e-CPF, NF-e),
> acessos a sites com manual ilustrado e alvarás — organizados por empresa,
> com permissão por setor e aviso de vencimento.

Código em: `~/Dev/cofre-digital`

## Estado atual

Rodando em Docker, acessível na rede em `http://<ip>:4004`. Três módulos de
conteúdo (certificados, acessos, alvarás) mais empresas, equipe e configurações.
Em jul/2026 passou por uma rodada de acabamento do design e, depois, pela
regularização da infra (ver abaixo).

## Infra (regularizada em 20/07/2026)

Primeiro projeto a receber o chassi do [[Infra]] por inteiro. Antes estava fora dele
em quase tudo: containers com sufixo do compose (`cofre-digital-db-1`), porta
`4004:3000` fixa no compose, `"dev": "next dev"` sem porta, volume `pgdata` genérico
e a porta do banco escondida num `docker-compose.override.yml` gitignorado — que
ainda por cima divergia do `.env.example` (5435 contra 5433).

Como ficou:

| Recurso | Nome | Porta no host |
|---|---|---|
| App | `cofre-digital-app` | `0.0.0.0:4004` → 3000 |
| Banco | `cofre-digital-db` | `127.0.0.1:5004` → 5432 |
| Migrations | `cofre-digital-migrate` | — |
| Rede | `cofre-digital-net` | — |
| Volume | `cofre-digital-db-data` | — |

O override sumiu: o bind `127.0.0.1` no compose principal dá o mesmo acesso local sem
expor o banco na rede e sem um segundo arquivo fora do repositório. Um compose só,
igual em dev e em produção — [[Ambiente de dev sobe igual ao de produção]].

A migração do volume (`cofre-digital_pgdata` → `cofre-digital-db-data`) foi feita com
`pg_dump` antes e cópia por container intermediário; os 5 certificados e 5 empresas
seguiram intactos. O volume antigo ficou parado como rede de segurança.

## O que tem

- **Certificados**: upload de `.pfx` com leitura automática do próprio arquivo
  (titular, CNPJ/CPF, tipo, AC, validade), A1 vs A3, download, anel de
  vencimento e histórico de eventos.
- **Acessos**: site, tipo de login, credenciais e **manual em Markdown** com
  imagens (upload ou Ctrl+V), com prévia ao escrever.
- **Alvarás**: vencimento opcional (existe alvará permanente), arquivo anexo e
  histórico. Os sem data entram no total mas ficam fora das barras de situação.
- **Empresas**: cada uma com seu próprio cofre, agregando os três módulos.
- **Bloqueio do cofre**: PIN, bloqueio manual e automático por inatividade —
  camada extra além do login.
- **Janela de "vencendo" configurável**, valendo para badges, filtros e dashboard.

## Decisões importantes

- **Permissão validada na API, não escondendo botão.** A senha nem chega ao
  navegador dos setores de leitura: só o endpoint de cópia a entrega.
- **Perfis de permissão por módulo** (`view`/`edit`), em vez de amarrar tudo ao
  setor — o setor virou rótulo, o perfil virou a regra.
- **Sem biblioteca de UI.** Design system próprio com prefixo `vlt-` em
  `globals.css`, seguindo o [[Design]]. Coerente com
  [[Manter o tooling enxuto e o conhecimento no cérebro]].
- **Dark-first**: o cofre nasce no modo noturno, claro é opcional — o contrário
  do padrão dos outros projetos.
- **Preferências pessoais no navegador** (tema, PIN, janela de alerta), dados no
  Postgres. Ninguém precisa sincronizar gosto pessoal entre máquinas.

## Rodada de design (jul/2026)

Três buracos entre o app e o sistema do [[Design]], todos fechados:

1. **24 `alert()` nativos** eram o único feedback do app — janela do sistema,
   fora da linguagem visual, e só para erro. Viraram toast, e as ações de
   sucesso passaram a confirmar. Ver [[Toast em vez de alert para o feedback do app]].
2. **Carregamento em bloco cinza** (19 retângulos pulsando, e no dashboard dois
   vãos em branco). Virou esqueleto com a forma do conteúdo. Ver
   [[Esqueleto de carregamento imita a forma do conteúdo]].
3. **Filtro fora da URL** — a página de certificados chegava a *limpar* a query
   string. Agora busca e filtro moram na URL. Ver [[Filtro de lista mora na URL]].

## Stack

Next.js 16 (App Router) · React 19 · TypeScript · Tailwind v4 · Prisma 7 ·
PostgreSQL 17 · Docker Compose. Ícones lucide, PKCS#12 lido no navegador.

## Aprendizados (viraram notas atômicas)

- [[Toast em vez de alert para o feedback do app]]
- [[Esqueleto de carregamento imita a forma do conteúdo]]
- [[Filtro de lista mora na URL]]
- [[Portal condicional dispensa o flag de montagem]]
- [[NoInfer faz o genérico sair da lista, não do valor padrão]]

## Próximos passos possíveis

- Dropdown de filtro por empresa nas listas (o padrão com busca no topo do
  [[Padrões de componentes de dashboard]] ainda não foi usado aqui).
- Skeleton no refetch em vez de só no primeiro load, se o volume crescer.
- Exportar a lista filtrada (o filtro já está na URL, então o link já serve).

## Conexões

- Usa: [[Design]]
- Usa: [[Infra]]
- Faz parte de: [[Projetos]]
- Mapa: [[Projetos]]
