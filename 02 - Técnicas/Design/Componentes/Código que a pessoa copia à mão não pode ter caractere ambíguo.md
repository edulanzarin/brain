---
tags: [tipo/atomica, camada/padrao, design, armadilha]
criado: 2026-08-30
---

# Código que a pessoa copia à mão não pode ter caractere ambíguo

> Todo código que sai de uma tela e volta por outro meio — copiado à mão,
> ditado por telefone, escrito num papel e filmado — atravessa pelo menos uma
> etapa onde a forma da letra é tudo o que resta. Nessa etapa `0` e `O` são o
> mesmo desenho, e `1`, `I` e `l` também.

## As duas metades

**No sorteio**, tire os ambíguos do alfabeto. Sobram 31 caracteres, que é mais
que suficiente:

```ts
const ALFABETO = "23456789ABCDEFGHJKMNPQRSTUVWXYZ";  // sem 0/O, sem 1/I/L
```

**Na exibição**, tamanho de título e não de corpo. Um código de 13px numa tela
de celular é onde o `B` vira `8` na caneta — e o erro não aparece ali: aparece
depois, como uma recusa que a pessoa não tem como entender, porque ela copiou o
que viu.

## Por que doer mais do que parece

O caractere ambíguo não gera erro, gera **recusa de quem fez tudo certo**. Ela
refaz o trabalho inteiro, erra de novo pelo mesmo motivo, e a conclusão razoável
que ela tira é que o sistema não funciona. Nenhum log registra isso.

Vale para código de verificação, cupom, chave de licença, id que o suporte pede
por telefone, e qualquer coisa que alguém vá ler em voz alta.

## Conexões
- Princípio: [[A variante de um controle muda a intenção, não o tamanho]] — a
  exceção que confirma: aqui o tamanho é semântico, porque a peça existe para
  ser transcrita, não lida.
- Irmã: [[Artefato prova existência; só um desafio prova o momento]]
- Irmã: [[Glifo miúdo é lido como o símbolo mais próximo que a pessoa já conhece]]
- Visto em: [[Privello]]
- Mapa: [[Design]]
