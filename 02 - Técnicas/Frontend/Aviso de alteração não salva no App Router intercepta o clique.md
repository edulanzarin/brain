---
tags: [tipo/atomica, camada/padrao, dev/frontend, nextjs, armadilha]
criado: 2026-09-25
---

# Aviso de alteração não salva no App Router intercepta o clique

> O `beforeunload` pergunta "sair sem salvar?" quando a página vai descarregar:
> fechar a aba, recarregar, ir para outro site. A navegação do App Router não
> descarrega nada, troca a tela no mesmo documento, e o evento nunca dispara.
> Um editor que só confia nele perde as alterações no primeiro clique na barra
> lateral.

## A técnica

Dois ouvintes, porque são dois caminhos de saída:

1. **`beforeunload`** para o que sai do documento. Só aceita a pergunta nativa do
   navegador; `preventDefault()` e `returnValue = ""` para os antigos.
2. **`click` na captura da `window`** para os links do próprio app. Na captura,
   porque o `Link` do Next navega no `onClick` dele: parar o evento na fase de
   bolha é tarde. O ouvinte acha o `<a href>` mais próximo, e se for navegação
   interna, cancela, guarda o destino e abre o modal próprio. "Sair" faz o
   `router.push` do destino guardado.

O que o ouvinte deixa passar:

- Ctrl, Shift, Alt ou Meta com o clique, `target="_blank"`, `download`: abre em
  outra aba e o editor continua aqui.
- Outra origem: o `beforeunload` cuida.
- O mesmo caminho e a mesma query: não é saída.

O estado "sujo" vai numa ref lida pelos ouvintes, registrados uma vez só:
re-registrar a cada tecla digitada troca ouvinte no meio de um clique.

## O que continua escapando

- O botão Voltar do navegador (o App Router não expõe um bloqueio de
  navegação pelo histórico).
- `router.push` disparado por código, como a paleta de comandos.

Quem precisar desses dois pede um bloqueio no próprio roteador, ou um estado que
sobreviva à saída (rascunho guardado).

## Conexões
- Irmã: [[router.replace do Next falha no build de produção]]
- Visto em: [[NaveX]]
- Mapa: [[Frontend]]
