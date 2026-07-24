---
tags: [tipo/atomica, camada/principio, dev/git]
criado: 2026-07-23
---

# Versão é corte deliberado em SemVer, não efeito de cada merge

> Todo trabalho vive numa branch e volta pra `main` por merge; a **tag de versão** (SemVer `vX.Y.Z`) sai quando eu fecho um ciclo, não automaticamente a cada merge.

## A regra

O fluxo tem duas velocidades diferentes, e elas não se confundem:

- **Branch + commit + merge** é a velocidade do trabalho. Cada coisa nasce numa branch nomeada pelo tipo (`feat/*`, `fix/*`, `chore/*`, `refactor/*`), commitada em Conventional Commits (`feat(escopo):`, `fix(escopo):`…), e volta pra `main` por merge quando está pronta. Isso acontece o tempo todo.
- **Tag de versão** é a velocidade do release. A `main` acumula vários merges e a tag `vX.Y.Z` só é criada quando **eu** decido que fechou um ciclo entregável. Não é uma tag por merge.

SemVer diz a natureza do que mudou desde a última tag:

- **MAJOR** (`v1.0.0`) — quebra: algo que já funcionava mudou de contrato.
- **MINOR** (`v0.2.0`) — feature nova, retrocompatível. Some `feat:` desde a última tag.
- **PATCH** (`v0.1.1`) — só correção, sem feature. Some `fix:` desde a última tag.

Antes de `1.0.0` o app ainda está se firmando: MINOR carrega feature e pode carregar pequena quebra sem cerimônia.

## Por que

Duas dores que isso resolve:

- **Tag por merge vira ruído.** Se cada feature que entra na `main` vira versão, a lista de tags cresce sem contar história nenhuma — vira log de merge duplicado. Corte deliberado faz cada `vX.Y.Z` marcar um estado que valeu a pena poder voltar.
- **O número tem que falar.** `v0.2.0` diz "ganhou capacidade", `v0.2.1` diz "consertou". Quem olha o histórico entende a gravidade sem abrir o diff. É assim que o kernel do Linux corta versão: acumula na árvore e fecha a versão num ponto escolhido, não a cada patch aceito.

O que **não** é opinião livre: trabalho grande ou arriscado nunca vai direto na `main` — sempre branch, é a rede de segurança que deixa desfazer. Casa com [[Antes de mexer, o trabalho anterior já tem que estar commitado]] se existir; senão, é a mesma raiz: o histórico é o desfazer.

## Na prática

```
feat/folha    → conventional commits → merge na main
feat/admin    → conventional commits → merge na main
fix/balancete → conventional commits → merge na main
                     ↓  eu decido: fechou o ciclo
              git tag -a v0.2.0 -m "..."   (anotada, não lightweight)
              git push origin v0.2.0
```

- Tag **anotada** (`-a`), nunca lightweight — carrega autor, data e mensagem.
- `package.json` `version` acompanha a tag no mesmo corte.
- Merge na `main` por fast-forward quando a branch está à frente sem divergência; o histórico fica linear e legível.

## Conexões
- Irmã: [[Verificar no build de produção, não só em dev]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Base]]
