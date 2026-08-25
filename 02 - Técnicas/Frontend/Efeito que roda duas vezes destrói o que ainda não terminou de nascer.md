---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-08-25
---

# Efeito que roda duas vezes destrói o que ainda não terminou de nascer

> Em `useEffect` que cria um recurso por caminho assíncrono, só publique a
> referência **depois** que a criação terminar. Em desenvolvimento o React roda
> efeito, limpeza e efeito de novo; a limpeza da primeira passada chega enquanto
> a criação ainda está no ar, e destruir um objeto meio-construído estoura em
> lugares que não têm nada a ver com o seu código.

## O problema

O padrão que parece certo guarda a referência antes de terminar:

```tsx
useEffect(() => {
  let app: Application | null = null;

  (async () => {
    app = new Application();      // <- publicado cedo demais
    await app.init({ ... });
    // ...
  })();

  return () => app?.destroy();     // roda com o init no meio do caminho
}, []);
```

A limpeza pega o objeto entre o construtor e o init. No PixiJS v8 isso aparece
como `this._cancelResize is not a function`, porque os plugins do `Application`
são instalados no `init`, e o `destroy` conta com eles. Cada biblioteca falha do
seu jeito, e nenhuma dessas mensagens aponta pra causa.

O que engana é o modo: em produção o efeito roda uma vez e nada quebra. O bug só
existe em dev — que é onde você passa o dia.

## A solução

Construa numa variável **local**, publique na variável de fora só depois do
`await`, e trate o cancelamento no meio do caminho:

```tsx
useEffect(() => {
  let app: Application | null = null;
  let cancelled = false;

  (async () => {
    const local = new Application();
    await local.init({ ... });
    if (cancelled) {
      local.destroy(true, { children: true });  // nasceu inteiro, morre inteiro
      return;
    }
    app = local;                                 // só agora existe pra limpeza
    // ...
  })();

  return () => {
    cancelled = true;
    app?.destroy(true, { children: true });
    app = null;
  };
}, []);
```

A regra em uma frase: **a limpeza só pode ver o que já nasceu inteiro**.

## O que mais vale lembrar

Vale para qualquer criação assíncrona em efeito: WebSocket, `AudioContext`,
observer de mídia, worker, mapa. O sinal de que você está nesse caso é ter um
`await` entre a criação e o uso.

Isso não se acha lendo o código — se acha abrindo a tela. Ver
[[Verificar no build de produção, não só em dev]] para o espelho deste caso.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[Mundo imperativo e React se falam por eventos, não por referência]]
- Visto em: [[Vespéria]]
- Mapa: [[Frontend]]
