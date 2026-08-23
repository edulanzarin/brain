---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-08-23
---

# O empacotador segue o valor importado, não o tipo

> Um componente de cliente pode importar `type` de qualquer módulo do servidor: o
> tipo é apagado na compilação e nada vai junto. Importar **um valor** do mesmo
> arquivo — uma função, uma constante, um estado inicial — arrasta o módulo
> inteiro e as dependências dele pro bundle do navegador.

## O sintoma

Um build que compilava havia semanas quebrou com sete erros seguidos:

```
Module not found: Can't resolve 'dns'
Module not found: Can't resolve 'fs'
Module not found: Can't resolve 'net'
Module not found: Can't resolve 'tls'
```

Nenhum desses nomes aparecia no código. Eles são dependências do driver do
Postgres, que o navegador não tem. O caminho até lá tinha quatro saltos:

```
painel-tool.tsx ("use client")
  → motor/sessao.ts        (estadoParado)     ← import de VALOR
    → vinculo.ts
      → db.ts
        → pg → net, tls, dns, fs
```

O componente importava dois tipos e **uma função**: `estadoParado()`, usada como
valor inicial do `useState`. Os tipos não custavam nada; a função custou o
processo inteiro.

## A saída

O contrato entre servidor e tela vira módulo próprio, sem nenhum import pesado:

```
motor/tipos.ts    interfaces + estadoParado()  ← cliente e servidor importam
motor/sessao.ts   a classe, o banco, o socket  ← só servidor
```

`sessao.ts` reexporta `tipos.ts`, então o servidor continua importando de um
lugar só e nada muda pra ele.

## Como reconhecer antes de quebrar

- A mensagem **nunca cita o seu arquivo** — ela cita `net` e `fs`. Comece pelo
  módulo de cliente mais recente e siga os imports de valor.
- `import type { X }` é sempre seguro. `import { X }` só é seguro se `X` for
  usado apenas em posição de tipo, e é fácil deixar de ser numa refatoração.
- O sinal precoce é conceitual: se um arquivo exporta **o formato do dado** e
  **quem produz o dado**, ele tem dois públicos e vai ser importado pelos dois.

## Conexões
- Princípio: [[Verificar no build de produção, não só em dev]]
- Irmã: [[Componente de ícone não atravessa a fronteira server-client]] ·
  [[Componente de terceiro que usa Context não roda em Server Component]]
- Visto em: [[piwdex2]]
- Mapa: [[Frontend]]
