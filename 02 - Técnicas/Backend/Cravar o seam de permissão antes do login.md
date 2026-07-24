---
tags: [tipo/atomica, camada/padrao, seguranca, dev/backend]
criado: 2026-07-20
---

# Cravar o seam de permissão antes do login

> Antes de existir autenticação, crie a costura por onde a permissão passa —
> uma `getSessao()` stub — e faça todo o resto já perguntar a ela. Quando o
> login chegar, só o stub muda; nada precisa ser retrofitado.

## O problema que evita

Permissão adicionada depois vira caça: cada rota, cada tela, cada action tem que
ser revisitada pra enfiar uma checagem, e a que você esquecer fica aberta. O
custo não é a regra de permissão — é o retrofit espalhado.

## A costura

Uma função só concentra "quem é o usuário e o que ele pode":

```ts
// hoje: stub. amanhã: lê cookie de sessão + banco. A assinatura não muda.
export const getSessao = cache(async (): Promise<Sessao> => {
  return { usuario: { id: "dev", ... }, modulos: { fiscal: "edit", contabil: "edit" } };
});
```

Tudo consulta ela desde o dia um: a tela que decide o que mostrar, o layout que
barra a entrada, a rota que serve o dado. Enquanto o stub libera tudo, o app
funciona igual — mas os pontos de checagem já existem e estão exercitados. O
login vira preencher `getSessao()`; o grafo de chamadas já está pronto.

## O gate mora num lugar só, derivado de convenção

O ponto mais fácil de esquecer é a rota de API. Em vez de repetir a checagem em
N rotas (e esquecer na de número N+1), **derive o alvo da convenção de caminho**
e cheque no wrapper único das rotas:

```ts
// /api/fiscal/... e /api/contabil/... dizem o módulo pelo próprio path
const modulo = req.nextUrl.pathname.match(/^\/api\/(fiscal|contabil)\b/)?.[1];
if (modulo && !(await podeAcessar(modulo, req.method === "GET" ? "view" : "edit")))
  return Response.json({ error }, { status: 403 });
```

Assim nenhuma rota nova nasce desprotegida — o gate é estrutural, não uma linha
que o autor precisa lembrar de colar. Ler exige `view`, escrever exige `edit`.

## Otimista na tela, real no dado

Duas checagens com papéis distintos (a regra de fundo em
[[Permissão se valida no servidor, não na interface]]): o layout/tela checa pra
esconder e redirecionar (conveniência, e não re-roda em navegação client-side); o
wrapper da rota **nega de fato**, a cada requisição, perto do dado. A da tela some
o botão; a do servidor tranca a porta.

## Conexões
- Princípio: [[Permissão se valida no servidor, não na interface]]
- Depende de: [[Configuração vem do ambiente, não do código]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Backend]]
