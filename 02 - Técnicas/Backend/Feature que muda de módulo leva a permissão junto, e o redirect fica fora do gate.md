---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-09-03
---

# Feature que muda de módulo leva a permissão junto, e o redirect fica fora do gate

> Mover uma tela de um módulo para outro parece mudança de pasta. Não é: ela
> desliga o acesso das pessoas que só entravam ali por causa daquela tela, e o
> endereço antigo continua circulando na cabeça de quem usava.

## O problema

Num app com permissão por seção (`modulo/secao`), a tela mudar de casa muda a
chave da permissão. Quem tinha `folha/post-mortem` não tem `postmortem/dp`: no
primeiro login depois do deploy a pessoa não perde uma tela, perde **o acesso**,
e a leitura dela é "o sistema quebrou".

A armadilha fica na segunda camada. O reflexo é deixar uma página de
redirecionamento no caminho antigo — e ela nunca roda para quem mais precisa
dela. O caminho antigo está debaixo do layout do módulo antigo, e esse layout
começa exigindo acesso a alguma seção daquele módulo. Justamente quem só tinha a
tela que saiu deixou de ter qualquer seção lá: o gate manda essa pessoa para o
launcher antes de a página existir. A rota de redirecionamento funciona ao ser
testada por quem tem o módulo inteiro (o admin), e falha em silêncio para o
público dela.

## A solução

Duas coisas, na mesma mudança:

**1. A migration converte a permissão**, e converte com significado — o par
antigo vira o par novo que corresponde ao que a pessoa fazia, não ao nome
parecido:

```sql
-- quem preenchia o do DP continua preenchendo o do DP
update cargo_secao set modulo = 'postmortem', secao = 'dp'
 where modulo = 'folha' and secao = 'post-mortem';
-- quem via TODOS os do DP passa a ver o escritório inteiro
update cargo_secao set modulo = 'postmortem', secao = 'geral'
 where modulo = 'folha' and secao = 'post-mortem-gestao';
```

**2. O redirecionamento sobe para o middleware** (`proxy.ts` no Next 16), que
roda antes de qualquer layout e não depende de permissão nenhuma:

```ts
function mudouDeCasa(pathname: string): string | null {
  if (pathname === "/folha/post-mortem-gestao") return "/post-mortem/geral";
  if (pathname === "/folha/post-mortem") return "/post-mortem/dp";
  const item = pathname.match(/^\/folha\/post-mortem\/(\d+)$/);
  return item ? `/post-mortem/dp/${item[1]}` : null;
}
```

Vale o mesmo para o link antigo de um registro: o id não muda, só o caminho, e
mandar o `/(\d+)/` junto é o que evita transformar "abre o relatório 12" em 404.

## O que mais vale lembrar

- **O teste que revela é o de quem tem pouco acesso.** Conferir a mudança com o
  usuário admin não prova nada: ele passa por todo gate. O caso interessante é o
  perfil que só tinha a tela que se moveu.
- **Renomear a tabela não renomeia o que veio com ela.** No Postgres, `alter
  table ... rename` deixa pkey, unique e sequência com o nome velho, e o nome
  velho volta a aparecer na próxima mensagem de erro. Renomeie índices,
  sequências e trigger no mesmo passo.
- **Coluna nova em linha existente: default para o backfill, e o default sai
  logo depois.** `add column setor text not null default 'dp'` carimba o que já
  existe; `alter column setor drop default` impede que a próxima inserção
  esquecida vire "DP" em silêncio.
- **A lista fixa que ninguém lembra de atualizar.** Um módulo novo costuma
  esbarrar em algum allowlist escrito à mão — no caso, o da trilha de auditoria,
  que recusava o registro da exportação e deixava o CSV sair sem rastro.

## Conexões
- Princípio: [[Identificador que já circulou não é mais seu para mudar]]
- Irmã: [[Posse numa permissão binária é duas seções e recorte por linha]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Backend]]
