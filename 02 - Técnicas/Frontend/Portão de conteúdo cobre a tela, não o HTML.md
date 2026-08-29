---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-08-29
---

# Portão de conteúdo cobre a tela, não o HTML

> Confirmação de maioridade, aviso de cookie, "aceite os termos para continuar":
> o portão é uma camada **por cima** da página já renderizada. Se ele decidir o
> que o servidor manda, o buscador lê um site vazio e a página deixa de existir.

## O erro

O reflexo é tratar o portão como roteamento: sem o cookie, o servidor devolve a
tela de confirmação **no lugar** da página.

```tsx
// errado
if (!jar.get(COOKIE)) return <PortaoIdade />;
return <Pagina />;
```

Funciona para a pessoa e destrói o site. O robô do buscador não tem cookie,
nunca vai ter, e não clica em botão nenhum: para ele, **toda** URL do domínio
devolve a mesma tela de confirmação. Conteúdo duplicado em cada endereço, e
nenhum conteúdo em endereço nenhum.

## A forma

O portão entra no layout, depois do conteúdo, como overlay:

```tsx
<body>
  <Cabecalho />
  <main>{children}</main>
  <Rodape />
  {/* já veio tudo renderizado; isto só cobre */}
  <PortaoIdade />
</body>
```

`PortaoIdade` é componente de servidor: lê o cookie e devolve `null` se ele
existe. Quem não tem cookie recebe um `position: fixed` com fundo opaco por cima
de uma página que **está inteira no HTML**.

## O que mais vale lembrar

- **O que o portão protege é a vista, não o dado.** Ele não é controle de acesso:
  qualquer um remove o overlay pelo devtools. Conteúdo que precisa mesmo de
  trava não pode estar no HTML — ver
  [[Permissão se valida no servidor, não na interface]]. As duas coisas convivem
  na mesma página: o portão cobre tudo, e o telefone de quem só atende assinante
  não chega ao HTML de quem não assina.
- **A saída é um link de verdade** (`<a href>` para fora), não um botão que fecha
  nada. Quem clica em "sair" precisa sair.
- Site adulto ainda quer a meta `rating` do RTA no `<head>`: é o que os filtros
  parentais leem. O portão não substitui esse rótulo.

## Conexões
- Princípio: [[Permissão se valida no servidor, não na interface]]
- Irmã: [[A superfície indexável sai da mesma consulta que o conteúdo]]
- Visto em: [[Privello]]
- Mapa: [[Frontend]]
