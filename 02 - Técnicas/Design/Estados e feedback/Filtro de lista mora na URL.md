---
tags: [tipo/atomica, camada/padrao, design, dev/frontend]
criado: 2026-07-20
---

# Filtro de lista mora na URL

> Se o usuário não consegue mandar a lista filtrada para alguém, o filtro está
> preso no componente errado.

## O quê

Busca e filtro viram query param, sincronizados com `history.replaceState`:

```
/certificados?q=teste&status=expiring
```

Detalhes que fazem funcionar:

- **`history.replaceState`, não `router.replace`** — o do Next quebra no build
  de produção (ver [[router.replace do Next falha no build de produção]]) e ainda
  remontaria a página a cada tecla digitada na busca.
- **Valor padrão sai da URL.** `status=all` não vira parâmetro; link limpo
  enquanto nada foi filtrado.
- **Ler a URL depois de montar**, não no inicializador do `useState`: o HTML do
  servidor não enxerga a query, e divergir disso quebra a hidratação. Custa um
  frame com o valor padrão.

## Por que importa

É o princípio 4 do [[Design]] ("estado na URL"), e o [[Cofre Digital]] fazia o
**oposto**: a página de certificados lia os parâmetros de deep-link e então
limpava a query string inteira. Recarregar perdia a busca, e não dava para
mandar "olha os que vencem esse mês" para ninguém.

Ao consertar, separar os dois papéis do query param ficou explícito:

- **Deep-link** (`?novo=1`, `?cert=id`) é comando: consome e limpa.
- **Filtro** (`?q`, `?status`) é estado: fica.

O bug era o primeiro papel levando o segundo junto na limpeza.

## Conexões
- Princípio: [[Estado compartilhável mora na URL]]
- Ver também: [[router.replace do Next falha no build de produção]] · [[Padrões de componentes de dashboard]]
- Visto em: [[Cofre Digital]]
- Faz parte de: [[Design]]
- Mapa: [[Design]]
