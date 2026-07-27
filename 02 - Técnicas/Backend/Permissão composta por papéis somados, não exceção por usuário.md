---
tags: [tipo/atomica, camada/padrao, dev/backend, sql, armadilha]
criado: 2026-07-27
---

# Permissão composta por papéis somados, não exceção por usuário

Quando o acesso de uma pessoa começa a ganhar "ajuste fino" — libera esta seção só
pra fulano, bloqueia aquela, soma uma empresa avulsa —, o modelo vira **exceção por
gente**: a permissão se espalha por várias tabelas e ninguém mais sabe o que fulano
enxerga sem reconstruir na cabeça. É trabalhoso e inconsistente, e cada usuário novo
repete o mesmo conjunto na unha.

O modelo que se sustenta: **o PAPEL (cargo) concentra toda a permissão** — as seções,
o escopo de empresa e até os flags de acesso amplo (é admin? vê todas as empresas?). A
pessoa **acumula um ou mais papéis** e o acesso efetivo é a **UNIÃO** de tudo que eles
concedem. Precisa de algo diferente? Cria/atribui outro papel (inclusive um papel
especial), em vez de abrir exceção na pessoa.

O que faz valer a pena:

- **Uma fonte só de verdade.** Pra saber o que alguém acessa, olha os papéis dela —
  nada de permissão mora no usuário.
- **"Admin" deixa de ser flag do usuário** e vira um papel com acesso total, criado
  idempotente no seed e vinculado ao usuário admin.
- **Aditivo é natural com N:N + union**: uma tabela `usuario_papel` (N:N) e a sessão
  resolve com `where papel_id = any($papeis)` + `distinct`. Não existe "remover" um
  privilégio que o papel dá — se não deve ter, é outro papel. É isso que diferencia do
  modelo com *override* por usuário, onde a exceção ainda existe (e é a armadilha).
- **Empresa avulsa por pessoa some**: se precisa de uma empresa específica, ela entra
  num grupo reutilizável, e o grupo entra num papel.

Migração do modelo antigo (um papel + tabelas de override) pro novo: cria a N:N, copia
o papel único pra ela, e as tabelas de exceção por usuário ficam mortas (dropar depois).
Um "papel especial" cobre quem tinha exceção legítima. A união é barata de calcular e
fácil de explicar — vale mais que a flexibilidade que o override dava e quase ninguém
usava direito.

## Conexões
- Princípio: [[Permissão se valida no servidor, não na interface]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Backend]]
