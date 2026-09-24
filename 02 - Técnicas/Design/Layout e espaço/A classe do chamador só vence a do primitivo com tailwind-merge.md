---
tags: [tipo/atomica, camada/padrao, dev/frontend, design]
criado: 2026-08-13
---

# A classe do chamador só vence a do primitivo com tailwind-merge

> Um primitivo que aceita `className` de fora e tem defaults em **utilitário**
> (`p-5`, `h-9`, `rounded-lg`) não pode só concatenar as classes. `cn("p-5",
> "p-4")` deixa as duas no atributo e quem vence é a ordem no CSS gerado, não a
> intenção do chamador. `tailwind-merge` resolve o conflito: a última — a de quem
> chamou — ganha, de forma previsível.

## Quando concatenar basta e quando não basta

[[Primitiva de botão fecha o tamanho e abre só a variante]] junta as classes com
`[...].join(" ")` e funciona — mas por um motivo específico: lá a base mora em
`@layer components` ([[Classes de componente vão em @layer components no Tailwind]]),
então a **utilitária** que o chamador passa já vence a base por camada, sem precisar
de merge.

O merge vira necessário quando o default do primitivo é ele mesmo um **utilitário**:
um `Card` com `p-5` de padrão e um chamador que quer `p-4`. As duas são utilitárias,
mesma especificidade, mesma camada — só a ordem no arquivo decide, e isso é loteria.
`tailwind-merge` reconhece que `p-4` e `p-5` mexem na mesma coisa e descarta a antiga.

## A forma

```ts
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

`clsx` cuida do condicional (`cond && "classe"`, objetos, arrays); `tailwind-merge`
cuida do conflito. Todo primitivo então fecha em `cn(base, variante, className)`, e
o `className` de fora sobrepõe sem `!important` nem prop de escape.

Custa uma dependência (`tailwind-merge`) e um passe a mais por render — barato perto
de override inline espalhado. É a mesma ideia de garantir o comportamento na
estrutura, não na disciplina de quem usa.

## O merge não atravessa variante

Default **responsivo** no primitivo escapa do merge. No [[telebot]] (set/2026) a
superfície passou de `p-5` para `p-4 sm:p-5`, e todo chamador que pedia `p-0` (lista
encostada na borda) ficou com `p-0 sm:p-5`: o merge troca `p-4` por `p-0`, mas
`sm:p-5` é outro grupo e fica. No celular, certo; no desktop, a lista ganhava 20px
de recuo sem ninguém ter pedido.

Default que muda por tamanho de tela vai para classe de componente
(`@layer components`, com a media query dentro), e aí qualquer utilitária do
chamador vence em todos os tamanhos, pela camada.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Irmãs: [[Primitiva de botão fecha o tamanho e abre só a variante]] · [[Classes de componente vão em @layer components no Tailwind]]
- Visto em: [[Navetech Hub]] · [[telebot]]
- Mapa: [[Design]]
