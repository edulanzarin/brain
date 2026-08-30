---
tags: [tipo/atomica, camada/padrao, armadilha, dev/frontend]
criado: 2026-08-30
---

# A segunda ação do formulário se marca no botão, não no estado

> Um formulário com "salvar" e "apagar" precisa dizer ao servidor qual dos dois
> foi clicado. Limpar o campo no `onClick` do apagar não faz isso: o envio lê o
> campo de agora, e o `setState` só chega no render seguinte.

## O problema

O recado do perfil tinha um campo controlado e dois botões. O apagar parecia
óbvio:

```tsx
<Botao type="submit" onClick={() => setTexto("")}>Apagar</Botao>
```

O React agenda a mudança de estado e o envio acontece antes do próximo render,
então o servidor recebe o texto **antigo** e regrava exatamente o recado que a
pessoa mandou apagar. Nada dá erro: a ação roda, a tela recarrega, o recado
continua lá.

Formulário não aninha, então dois `<form>` também não resolvem. E pôr o botão
no rodapé do modal não funciona — o rodapé é irmão do corpo no HTML, e um
`submit` de fora do `<form>` não envia nada.

## A solução

O botão que envia carrega a intenção no próprio HTML, com `name` e `value`. O
navegador só inclui no `FormData` o botão que foi clicado.

```tsx
<Botao type="submit" name="apagar" value="1">Apagar</Botao>
<Botao type="submit" variante="rosa">Publicar</Botao>
```

```ts
if (dados.get("apagar") || !texto) { /* limpa */ }
```

Funciona com `useActionState` sem nenhum arranjo extra, e a intenção vira dado
do envio em vez de virar sequência de passos que precisa acontecer na ordem
certa.

## O que mais vale lembrar

O `form="id"` de HTML deixaria o botão morar no rodapé do modal e ainda enviar.
Vale quando o desenho pede o botão lá; custa um id inventado só para atravessar
uma parede — mais simples é o botão morar dentro do formulário.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]] — a
  intenção fica gravada na estrutura do envio, não num encadeamento de estado.
- Irmã: [[React reseta o formulário ao fim de uma Server Action]]
- Visto em: [[Privello]]
- Mapa: [[Frontend]]
