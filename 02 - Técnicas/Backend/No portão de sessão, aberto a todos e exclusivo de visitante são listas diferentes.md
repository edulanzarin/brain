---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-09-21
---

# No portão de sessão, aberto a todos e exclusivo de visitante são listas diferentes

> `/login` e uma página pública são as duas "sem autenticação", e por isso acabam na mesma lista do middleware. Só que o portão faz coisas opostas com elas: quem tem sessão é expulso de `/login` e precisa entrar na pública.

## O que acontece

O padrão comum nasce com uma lista só, quando `/login` é a única rota sem sessão:

```ts
const PUBLIC_PATHS = ["/login"];
// ...
if (!authenticated && !isPublic) redirect("/login");
if (authenticated && isPublic) redirect("/");   // a segunda regra é a armadilha
```

A segunda regra existe por um motivo legítimo: quem já está logado não deve ver a tela
de login. Mas ela está escrita contra `isPublic`, e não contra "é a tela de login".
Enquanto as duas coisas coincidem, ninguém percebe.

No dia em que entra uma página realmente pública, acrescentá-la a `PUBLIC_PATHS`
resolve o acesso de quem não tem conta **e quebra o acesso de quem tem**: o usuário
logado é chutado para a home ao abrir a página aberta. O sintoma é bizarro o bastante
para render meia hora de investigação — funciona na janela anônima e falha na normal.

## A correção

Separar os dois predicados, porque são dois:

```ts
const PUBLIC_PATHS = ["/publico", "/api/publico"];  // qualquer um, logado ou não
const GUEST_ONLY_PATHS = ["/login"];                // só quem não tem sessão

if (!authenticated && !isPublic && !isGuestOnly) redirect("/login");
if (authenticated && isGuestOnly) redirect("/");
```

São três estados de rota, não dois: exige sessão, dispensa sessão, recusa sessão.

## O que vem junto

Rota aberta não reusa a casca autenticada. A moldura do app (barra lateral, cadeado de
inatividade, hooks de "quem sou eu" e de configuração) vive de endpoints com sessão;
montada numa página sem login, cada um responde 401 e alguns empurram a tela de volta
para o login — a página pública fica inalcançável por um caminho que não é o portão.
A pública ganha a própria casca, mínima, e só reusa componentes que não buscam nada
sozinhos.

Um primitivo que lê configuração por hook é justamente o que atravessa essa fronteira
sem avisar. O conserto é o primitivo aceitar o valor de quem chama, mantendo o hook
como padrão — ver [[O primitivo só padroniza o que passa por dentro dele]].

## Conexões
- Princípio: folha isolada — nenhum princípio da base cobre isto ainda
- Depende de: [[Permissão se valida no servidor, não na interface]]
- Ver também: [[O primitivo só padroniza o que passa por dentro dele]]
- Visto em: [[Cofre Digital]]
- Mapa: [[Backend]]
