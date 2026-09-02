---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-09-02
---

# A preferência gravada não é o estado em vigor

> O botão de tema lia o `localStorage` para saber se a tela estava escura. Na
> primeira visita não há nada gravado — o tema veio da preferência do sistema
> operacional. Resultado: tela escura e botão oferecendo "usar tema escuro".

## O problema

Quem **aplica** o tema é um script que roda antes da primeira pintura:

```js
var t = localStorage.getItem("tema");
if (t !== "light" && t !== "dark") t = matchMedia("(prefers-color-scheme: dark)").matches ? "dark" : "light";
document.documentElement.dataset.theme = t;
```

Repare que o armazenamento é **uma das entradas**, não a saída. A saída — o que
de fato está valendo — é o atributo no `<html>`. Ler a entrada para descobrir a
saída funciona em todos os testes de quem já clicou no botão uma vez, e falha
exatamente para quem nunca clicou.

## A solução

Tratar o DOM como o que ele é: uma **fonte externa de estado**, lida por
`useSyncExternalStore`, que ainda tem um retrato próprio para o servidor.

```tsx
const tema = useSyncExternalStore(
  (aviso) => { window.addEventListener("nexo:tema", aviso);
               return () => window.removeEventListener("nexo:tema", aviso); },
  () => document.documentElement.dataset.theme === "dark" ? "dark" : "light",
  () => "light"          // servidor: o mesmo padrão do script
);
```

O `localStorage` continua sendo escrito — mas como **persistência da escolha**,
não como fonte de leitura.

## O que mais vale lembrar

- **O padrão do retrato de servidor tem que ser o mesmo do script**, senão o
  ícone pisca ao hidratar.
- Isto substitui o par `useEffect` + `setState` que se usa para "ler o que só
  existe no cliente". Aquele par funciona, mas provoca uma renderização extra a
  cada montagem, e o lint do React o recusa com razão — `useSyncExternalStore`
  entrega o valor certo na MESMA passada.
- A pergunta que generaliza: **quem escreve não é necessariamente quem sabe.**
  Antes de ler uma preferência, pergunte se ela é a única entrada do estado. Se
  houver um padrão do sistema, uma herança ou um cálculo no meio, a verdade está
  no ponto de aplicação.

## Conexões
- Princípio: [[Estado mutável se lê da fonte no uso, não de cópia guardada]]
- Irmã: [[Portal condicional dispensa o flag de montagem]] · [[Sistema de cores e tema do dashboard]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Frontend]]
