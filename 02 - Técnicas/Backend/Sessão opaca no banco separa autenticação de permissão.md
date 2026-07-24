---
tags: [tipo/atomica, camada/padrao, seguranca, dev/backend]
criado: 2026-07-23
---

# Sessão opaca no banco separa autenticação de permissão

> O cookie de login carrega só um **token opaco** (32 bytes aleatórios); quem é o
> usuário e o que ele pode vem de uma linha `sessao` no banco, lida a cada
> requisição. Assim a permissão é sempre a atual, o logout é imediato, e nenhum
> dado sensível anda no cookie.

## Por que opaca no banco, e não um JWT com claims

Um token assinado que carrega as permissões dentro (JWT) fica **velho**: mudou o
perfil do usuário, o token antigo ainda diz o que ele podia antes, até expirar. E
não dá pra revogar sem uma lista de bloqueio — que já é um estado no servidor, o
que o JWT prometia evitar.

Com sessão no banco:
- **Autenticação** = "o token existe e não expirou" (a linha `sessao`).
- **Autorização** = lida do banco a cada request (`usuario` + perfil), então
  alterar uma permissão vale **na requisição seguinte**, sem deslogar ninguém.
- **Logout/revogação** = apagar a linha. Cookie roubado morre com ela ao expirar.
- Nada sensível no cookie: é um número aleatório, inútil fora do banco.

## O custo

Uma consulta ao banco por requisição para resolver a sessão. Barato, e memoizável
por request (no Next, `cache()` em volta do `getSessao`) para várias checagens na
mesma renderização compartilharem uma leitura só.

## Hash de senha sem dependência

`scrypt` do `node:crypto` basta — guardar no formato PHC `scrypt$N$salt$hash`
(com o custo N junto, pra evoluir sem quebrar hashes antigos) e comparar com
`timingSafeEqual`. Não precisa de biblioteca de auth pra login por email/senha.

## Conexões
- Princípio: [[Permissão se valida no servidor, não na interface]]
- Irmã: [[Cravar o seam de permissão antes do login]]
- Depende de: [[Configuração vem do ambiente, não do código]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Backend]]
