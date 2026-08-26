---
tags: [tipo/atomica, camada/padrao, dev/frontend, design]
criado: 2026-08-26
---

# Filtrar no cliente ou no servidor se decide pelo tamanho, não pelo gosto

> Duas listas filtráveis no mesmo site podem pedir soluções opostas. O que decide
> não é a stack nem a preferência por tela reativa: é **quanto dado precisa
> atravessar o fio pra a pergunta ser respondida**, e como esse tanto cresce.

## As duas escolhas

**Cliente** (estado em React, filtro em memória) quando o conjunto é pequeno e
tem TETO conhecido. A resposta sai na tecla, sem ida e volta, e o custo é uma vez
só, no carregamento.

**Servidor** (`<form method="get">`, filtro na rota) quando o conjunto é grande
ou cresce sem teto. O ganho não é performance de render: é não embarcar o acervo
inteiro em toda visita.

## O caso que mostra os dois

No [[piwdex2]], duas páginas irmãs, decididas ao contrário na mesma tarde:

| Página | Conjunto | Onde filtra |
|---|---|---|
| changelog do site | ~12 entradas escritas à mão, uma por trabalho meu | cliente |
| diário de patches do jogo | 1 patch = até 1.200 mudanças; o de 20/08 pesa 300 KB | servidor |

O changelog no servidor trocaria resposta instantânea por espera, para filtrar
um punhado de parágrafos. O diário no cliente exigiria mandar o acervo inteiro
em toda visita — e o custo cresceria a cada patch novo, ou seja, **a página
ficaria mais pesada exatamente porque o projeto deu certo**. Esse é o sinal mais
confiável de que o filtro está do lado errado.

## O que mais vale lembrar

- **Pergunte como o conjunto CRESCE, não quanto ele tem hoje.** Doze entradas que
  viram vinte continuam cabendo; um patch por semana não.
- O `<form method="get">` sai quase de graça e vem com três brindes que a versão
  reativa custa a ter: funciona sem JavaScript, é link compartilhável por
  construção, e o botão voltar faz o que a pessoa espera.
- **Um terceiro caso mora no meio**: conjunto grande já carregado na página por
  outro motivo (o catálogo que a tela toda usa). Aí filtrar no cliente é de
  graça, porque o dado já atravessou o fio.
- Filtro de cliente que escreve na URL usa `replace`, e não `push`: mexer num
  filtro não é navegar, e com `push` o botão voltar desfaz letra por letra o que
  foi digitado na busca. Ver
  [[router.replace do Next falha no build de produção]] pro sintoma que às vezes
  aparece aí (e para a medição que mostra que nem sempre aparece).
- Nos dois lados, a contagem exibida tem de ser a do RECORTE ATUAL. Chip que
  mostra o total do acervo ao lado de uma lista já filtrada é número que não
  descreve a tela — e número que não descreve a tela põe em dúvida o resto dela.

## Conexões
- Princípio: [[Peso de página se mede no fio, não na saída do render]]
- Irmã: [[Cache do React Query não é lugar de estado de interface]] ·
  [[Nota carrega só o que a pessoa não sabe]]
- Visto em: [[piwdex2]]
- Mapa: [[Frontend]]
