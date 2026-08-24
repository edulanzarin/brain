---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-08-24
---

# Trocar de sujeito na mesma rota não remonta, e o estado do anterior fica

> Quando uma tela passa a operar VÁRIOS sujeitos — várias contas, vários
> projetos, vários clientes —, trocar de um para outro costuma ser só mudar a
> query (`?conta=X`). A rota é a mesma, o componente é o mesmo, e o React o
> mantém montado: todo `useState` sobrevive à troca, e todo `useEffect(…, [])`
> não roda de novo.

## O que quebra, e por que não aparece

No [[piwdex2]] o painel do robô virou multi-conta. O servidor estava certo em
todas as rotas — cada escrita usa o id resolvido pelo portão. O vazamento era do
cliente:

- a **config** carregada da conta A continuava no `useState` ao abrir a B;
- o **rascunho** da aba de loja idem;
- as listas que cada painel busca no `useEffect` de montagem nunca eram relidas.

E o estrago passou do visual: o próximo "salvar" gravava os valores da conta A
**na conta B**. Sem erro nenhum, porque os dois lados são o mesmo tipo válido.
É o pior formato de bug numa tela cuja razão de existir é distinguir sujeitos.

## A correção é uma `key`, não uma lista de resets

```tsx
<PainelTool key={contaAtiva ?? "sem-conta"} … />
```

`key` diferente = árvore nova. Resolve de uma vez o estado de **todos** os
filhos, presentes e futuros — inclusive o do painel que alguém adicionar no mês
que vem.

A alternativa é um `useEffect` por pedaço de estado, e ela falha do jeito mais
caro: a lista fica incompleta no dia em que alguém adiciona um `useState` e não
lembra. Uma guarda que depende de memória não é guarda.

## O que mais vale lembrar

- **O sinal de alerta é a palavra "trocar de".** Trocar de conta, de empresa, de
  período, de idioma: se o sujeito muda e a rota não, pergunte o que sobrou.
- Vale para `EventSource`/WebSocket também: a assinatura antiga precisa entrar na
  lista de dependências, senão a tela recebe dois fluxos e pisca entre dois
  sujeitos sem nada indicar qual é qual.
- Um `useEffect` de reserva no estado mais perigoso ainda vale, para o dia em que
  alguém renderizar o componente sem a `key` — mas como segunda guarda, nunca
  como a primeira.

## Conexões
- Princípio: [[Estado de tela pertence à seção, não à página]]
- Irmã: [[Cache do React Query não é lugar de estado de interface]] ·
  [[Re-chavear um sistema é refactor mudo, force o compilador a achar as chamadas]] ·
  [[Estado compartilhável mora na URL]]
- Visto em: [[piwdex2]]
- Mapa: [[Frontend]]
