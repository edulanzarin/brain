---
tags: [tipo/atomica, design, dev/frontend, conceito, projeto/cofre-digital]
criado: 2026-07-20
---

# Toast em vez de alert para o feedback do app

> `alert()` é a única parte da tela que o design system não alcança. Todo app
> que tem design próprio precisa do seu canal de mensagem próprio.

## O quê

Um store minúsculo (Set de listeners + `useSyncExternalStore`, o mesmo formato
do toggle de tema) e um componente em portal montado no shell do app:

```
toast.success("Certificado guardado no cofre.")
toast.error("Sem permissão para editar.")
toastError(err, "Falha ao salvar.")   // atalho do catch
```

Decisões que valeram:

- **Cor só na faixa e no ícone**, fundo neutro. Toast de erro com fundo vermelho
  inteiro grita mais que o conteúdo.
- **Duração por tipo**: erro fica ~2x mais que sucesso — quem errou precisa ler.
- **Pilha limitada** (4): uma rajada de falhas não cobre a tela.
- **`aria-live="polite"`**: anuncia sem roubar o foco. Era exatamente o que o
  `alert()` fazia de pior — parava tudo e obrigava o clique.

## Por que importa

O ganho maior não foi trocar a caixinha: foi perceber que o app **só falava para
reclamar**. Todos os 24 `alert()` do [[Cofre Digital]] eram erro; salvar e
excluir não davam retorno nenhum. Um canal barato de mensagem faz o caminho
feliz ficar visível também.

## A armadilha: mensagem em dois lugares

Onde o **formulário já mostra o erro ao lado dos campos**, não toaste — a mesma
frase aparecendo inline e flutuando parece bug. Regra que ficou: erro de
validação de campo é do formulário; erro de operação (excluir, bloquear,
carregar) é do toast.

## Conexões
- Faz parte de: [[Design]]
- Aplicado em: [[Cofre Digital]]
- Ver também: [[Padrões de componentes de dashboard]] · [[Portal condicional dispensa o flag de montagem]]
