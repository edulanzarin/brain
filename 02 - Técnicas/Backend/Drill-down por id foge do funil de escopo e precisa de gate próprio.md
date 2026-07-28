---
tags: [tipo/atomica, camada/padrao, seguranca, dev/backend, sql]
criado: 2026-07-28
---

# Drill-down por id foge do funil de escopo e precisa de gate próprio

> O [[Escopo de dado se clampa no servidor, num funil só|funil de escopo]] protege
> as **listas** — as consultas que passam pelo `buildWhere`. Mas a rota de detalhe
> que recebe `?empresa=X&id=Y` e monta o próprio SQL **não passa pelo funil**. Ela
> tem só o gate de módulo (tem acesso ao Contábil?), nunca o de escopo (esta empresa
> é sua?). Cada drill-down precisa re-checar o dono do registro.

## O erro que evita

O gate de módulo autoriza o **endpoint**, não **qual registro** o usuário lê. Uma
rota de drill-down (itens de uma nota, balancete de uma empresa, plano de contas,
regras de um banco) costuma ler a empresa crua da query e consultar direto — o autor
confiou que "já está no módulo". Resultado: quem tem o módulo lê o dado de qualquer
empresa só trocando o `?empresa=` no DevTools. O funil não ajuda aqui porque essas
rotas nunca o chamam.

O tell no código: `const empresa = Number(sp.get("empresa"))` (ou `f.empresas[0]`)
seguido de `where codigoempresa = $1`, sem nenhuma checagem entre os dois.

## A costura

Um helper único, irmão do funil, chamado logo após ler o código da empresa:

```ts
// Barra empresa fora do alcance da sessão. Uma linha, greppável, em toda rota
// de empresa avulsa. Explícito é mais seguro que mágico para código de acesso.
export async function assertEmpresaVisivel(codigo: number) {
  const sessao = await getSessaoOpcional();
  if (!sessao || !podeVerEmpresa(sessao, codigo)) {
    throw new FilterError("Empresa fora do seu escopo de acesso");
  }
}
```

Auditar a cobertura é um grep: liste toda rota que extrai `empresa` avulsa e
confirme que cada uma ou passa por um funil ou chama o guard. O que sobrar é o furo.

O guard reusa o mesmo escopo do funil (`podeVerEmpresa` sobre a sessão), então
respeita a mesma regra de negócio: se o cargo do usuário libera aquela empresa, o
drill-down passa; senão, 403. Não é uma exclusão à parte — é o escopo do cargo
aplicado no ponto que o funil não alcança.

## Conexões
- Irmã: [[Escopo de dado se clampa no servidor, num funil só]] — o funil que este
  guard complementa; um cobre lista, o outro cobre o registro.
- Princípio: [[Permissão se valida no servidor, não na interface]]
- Parente: [[O que dois módulos compartilham é a query, não a rota]] — a query
  reusada por dois módulos também reusa (ou esquece) o gate de escopo.
- Visto em: [[Navetech Hub]]
- Mapa: [[Backend]]
