---
tags: [tipo/atomica, projeto/navedesk, dev/frontend, conceito]
criado: 2026-07-20
---

# React reseta o formulário ao fim de uma Server Action

> Quando uma `<form action={serverAction}>` termina, o React chama `reset()` no
> formulário — **dando certo ou errado**. Num erro de validação o usuário perde
> tudo que digitou.

## O caso concreto

No [[Navedesk]], abrir chamado com título curto demais devolvia
"O título precisa de ao menos 5 caracteres." e apagava junto a **descrição
inteira** que a pessoa acabara de escrever. O mesmo no cadastro: errar o
domínio do e-mail zerava os outros cinco campos.

Em campo de senha o reset é bem-vindo. Em texto longo é hostil.

## A correção

Controlar os campos que doem, com `useState`, e limpar só no sucesso:

```tsx
const [descricao, setDescricao] = useState("");
<textarea name="descricao" value={descricao}
          onChange={(e) => setDescricao(e.target.value)} />
```

Como o `estado` do `useActionState` muda a cada retorno, o re-render reaplica o
`value` e o texto sobrevive.

## A pegadinha do `<select>`

Com input de texto funciona; com `<select>` **não**. O `reset()` zera o DOM, e
como o state daquele campo não mudou, o React não reaplica o valor controlado
no re-render — o `<select>` volta para "Selecione…" sozinho.

Precisa de um empurrão explícito:

```tsx
const setor = useRef<HTMLSelectElement>(null);

useEffect(() => {
  if (setor.current && dados.setorId) setor.current.value = dados.setorId;
}, [estado, dados.setorId]);
```

## Como isso apareceu

Não apareceu em teste de unidade nem no typecheck — apareceu **dirigindo a tela
num navegador de verdade** e conferindo o `inputValue` depois do erro. É o
mesmo espírito de [[Verificar no build de produção, não só em dev]]: alguns
defeitos só existem no comportamento real.

## Links

- Descoberto em: [[Navedesk]]
- Ver também: [[Verificar no build de produção, não só em dev]]
- Faz parte de: [[Desenvolvimento]]
