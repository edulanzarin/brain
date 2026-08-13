---
tags: [tipo/atomica, camada/padrao, dev/frontend, design]
criado: 2026-08-08
---

# Primitiva de botão fecha o tamanho e abre só a variante

> Ter as classes `.btn`/`.icon-btn` no CSS não basta pra padronizar botão. O
> tamanho só fica firme quando o botão vira um **componente** que expõe
> `variant` (a cor) e **não** expõe `size` — aí o `!h-7` por instância não tem
> onde nascer. Aplicação de
> [[A variante de um controle muda a intenção, não o tamanho]].

## O sintoma

No [[navetalks]] existia um sistema de botões em CSS (`.btn`, `.btn-accent`,
`.btn-ghost`, `.btn-danger`, `.icon-btn`), mas a UI tinha "botão grande, botão
pequeno" na mesma tela. O motivo não era falta de sistema — era override por
instância espalhado:

```tsx
<button className="btn btn-ghost !h-7 !px-2 text-xs">Adicionar</button>
<button className="btn btn-accent">Editar</button>   {/* 36px, ao lado */}
<button className="btn btn-accent !w-9 !px-0"><Send/></button>
<span className="chip !h-4 !px-1.5 !text-[9px]">você</span>
```

Cada `!h-*`/`!w-*`/`!text-[*]` é uma micro-decisão de tamanho invisível isolada;
somadas, desalinham tudo.

## A técnica

Mova o controle pra uma primitiva onde **o tamanho não é parâmetro**. A variante
seleciona só a classe de cor; a forma é sempre a mesma.

```tsx
// Button: um tamanho (alvo 36px). A variante escolhe a COR, nunca o tamanho.
const VARIANT = { accent:"btn-accent", neutral:"", ghost:"btn-ghost",
                  danger:"btn-danger", wa:"btn-wa" } as const;

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  function Button({ variant="neutral", block, icon:Icon, className="", children, ...rest }, ref) {
    const cls = ["btn", VARIANT[variant], block?"w-full":"", className].filter(Boolean).join(" ");
    return <button ref={ref} className={cls} {...rest}>
      {Icon && <Icon size={16} />}{children}
    </button>;
  });
```

Regras que caíram junto:

- **Sem prop `size`** no botão e no icon-button. Quem precisaria de `!h-7` agora
  não tem a alavanca — o override sumiu por construção, não por combinado.
- **Ícone com tamanho fixo** dentro da primitiva (16 no Button, 18 no
  IconButton). O chamador passa o *componente* do ícone (`icon={Plus}`), não o
  tamanho.
- **IconButton exige `label`** (vira `aria-label`): botão só-ícone sem nome
  acessível é falha de HIG, então o tipo torna o nome obrigatório.
- **As classes-base vivem em `@layer components`** pra vencer a utilitária sem
  `!important` — [[Classes de componente vão em @layer components no Tailwind]].
- **Chip é rótulo, não controle**: pode ter duas escalas (`sm` inline / `md`
  solto), porque carrega informação, não ação. Botão de ação, não.

## O que sobra do lado do chamador

Vira declaração de intenção, sem número de tamanho nenhum:

```tsx
<Button variant="accent" icon={Plus}>Nova fila</Button>
<Button variant="danger" icon={Check}>Finalizar</Button>
<IconButton icon={Trash2} label="Excluir" />
<Chip tone="wa">Conectada</Chip>
```

No [[Navetech Hub]] (Nexo, ago/2026) a mesma primitiva nasceu de outro sintoma: três
"primário" concorrentes (gradiente, `bg-accent`, `bg-ink`) e dezenas de botões montados
à mão. Ali o `Button` também abre `size` (sm/md/lg fechados), porque a base tem controles
densos que precisam de altura menor — mas a instância nunca redefine `h-*` solto: escolhe
um dos tamanhos fechados. Como o default de tamanho é utilitário, o `className` de fora
precisa de [[A classe do chamador só vence a do primitivo com tailwind-merge]].

## Conexões
- Aplica: [[A variante de um controle muda a intenção, não o tamanho]] ·
  [[Escala fechada em vez de valor solto]]
- Usa: [[Classes de componente vão em @layer components no Tailwind]] ·
  [[A classe do chamador só vence a do primitivo com tailwind-merge]]
- Visto em: [[navetalks]] · [[Navetech Hub]]
- Mapa: [[Design]]
