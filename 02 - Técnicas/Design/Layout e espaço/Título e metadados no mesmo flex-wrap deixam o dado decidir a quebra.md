---
tags: [tipo/atomica, camada/padrao, design, armadilha]
criado: 2026-09-01
---

# Título e metadados no mesmo flex-wrap deixam o dado decidir a quebra

> Pôr o título e um grupo de selos/chips no mesmo container que quebra linha parece econômico — eles ficam juntos quando cabe e descem quando não cabe. O que acontece de verdade é que **o comprimento do título passa a decidir quantos selos sobem para a linha dele**, e esse número muda de registro para registro. Nome curto promove dois selos; nome longo promove um; e o promovido parece pertencer ao título, não ao grupo.

## O sintoma

Uma fileira de selos que "fica jogada": o primeiro colado no título, os outros
quebrando soltos embaixo em fileiras de larguras irregulares. Piora quando os
selos **não quebram por dentro** (`white-space: nowrap`, que é o certo para
selo): cada um é um bloco indivisível, então uma coluna estreita produz quase um
por fileira.

E ninguém consegue reproduzir de forma consistente, porque o defeito depende do
DADO — o mesmo layout parece bom no registro de nome curto e quebrado no de nome
longo.

## A regra

**Quebra de linha só dentro de um grupo, nunca entre grupos de papéis
diferentes.** O título é identidade; os selos são metadado sobre ela. Cada um no
seu container:

```jsx
<div className="flex flex-col gap-2">
    <h1>{nome}</h1>
    <div className="flex flex-wrap items-center gap-2">{selos}</div>
</div>
```

Assim o grupo de selos empacota sozinho, e a quebra passa a depender só da
largura disponível — que é a variável certa.

## A largura disponível é a outra metade do problema

No mesmo caso, o retrato dividia a linha com a ficha também no telefone: a ficha
ficava com ~290 px enquanto sobrava largura vazia embaixo do retrato. Empilhar os
dois abaixo do ponto de corte devolveu ~340 px, e a mesma lista de selos fechou
em duas fileiras cheias em vez de três ralas.

Vale medir antes de mexer no visual: some as larguras reais dos selos (padding +
ícone + gap + texto na fonte do rótulo) e conte quantas fileiras dão em cada
largura candidata. É aritmética, não gosto — e distingue "preciso de mais
largura" de "preciso de rótulo mais curto".

## Conexões
- Irmã: [[Fila de campos alinha por altura fixa de controle, não por items-end]] — o mesmo tipo de defeito, quando a fileira mistura coisas que não têm a mesma natureza
- Depende de: [[Trocar a fonte muda a largura, não só o desenho da letra]] — a conta das fileiras é feita na fonte de verdade
- Visto em: [[Privello]] — cabeçalho do perfil
- Mapa: [[Design]]
