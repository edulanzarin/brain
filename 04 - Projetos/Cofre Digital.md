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
Em jul/2026 passou por uma rodada de acabamento do design (ver abaixo).

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
