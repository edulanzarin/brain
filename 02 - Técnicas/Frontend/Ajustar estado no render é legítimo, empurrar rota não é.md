---
tags: [tipo/atomica, camada/padrao, armadilha, dev/frontend]
criado: 2026-08-30
---

# Ajustar estado no render é legítimo, empurrar rota não é

> O React documenta um padrão em que o componente chama `setState` durante o
> próprio render para se reajustar a uma prop que mudou. É tentador estender
> esse padrão para "e já que estou aqui, navego também". Não estende: ajustar o
> que é seu durante o desenho é legítimo, mexer no roteador não é, porque o
> roteador é outro componente.

## O aviso

```
Cannot update a component (`Router`) while rendering a different
component (`EditorDeExpediente`).
```

O código que provoca é este, e ele parece inocente porque a primeira metade é o
padrão oficial:

```tsx
const [estado, enviar] = useActionState(acao, undefined);

const [ultimo, setUltimo] = useState(estado);
if (estado !== ultimo) {
    setUltimo(estado);                              // legítimo: estado é meu
    if (estado?.ok && proximo) router.push(proximo); // não é: o Router é do vizinho
}
```

Nada quebra na tela. A navegação até acontece. O que o React está avisando é que
uma parte da árvore foi marcada para atualizar no meio do desenho de outra — e
esse é o tipo de coisa que funciona hoje, sobrevive ao StrictMode por sorte, e
um dia rende um render descartado no meio do caminho.

## A correção

A parte que é sua fica no render; a que é do vizinho vira efeito.

```tsx
useEffect(() => {
    if (estado?.ok && proximo) router.push(proximo);
}, [estado, proximo, router]);
```

Mesmo desfecho, mesma tela. A diferença é que agora a navegação acontece
**depois** que a resposta chegou e a tela já se desenhou, que é quando ela pode
acontecer sem atropelar ninguém.

## Como reconhecer antes do aviso

A pergunta é de quem é a coisa que está sendo mexida. `setState` do próprio
componente: dele. `router.push`, `revalidate`, `setState` de um contexto,
qualquer função que veio de um provider acima: não é dele. Só a primeira cabe
no render.

## Conexões
- Princípio: [[Estado de tela pertence à seção, não à página]] — a fronteira de
  posse é a mesma: durante o desenho, um componente só mexe no que é dele.
- Irmã: [[A segunda ação do formulário se marca no botão, não no estado]]
- Irmã: [[Efeito que roda duas vezes destrói o que ainda não terminou de nascer]]
- Visto em: [[Privello]]
- Mapa: [[Frontend]]
