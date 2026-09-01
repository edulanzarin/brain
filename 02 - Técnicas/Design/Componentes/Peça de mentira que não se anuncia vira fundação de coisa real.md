---
tags: [tipo/atomica, camada/padrao, design, armadilha]
criado: 2026-08-30
---

# Peça de mentira que não se anuncia vira fundação de coisa real

> Um botão "Denunciar" abria um modal com motivos, campo de texto e botão de
> enviar. Ao clicar, o modal fechava e uma torrada verde dizia "Denúncia
> enviada. A moderação vai olhar." Nada era escrito. Não havia ação, não havia
> rota, e a tabela nunca recebeu uma linha.

## Por que é pior que a peça faltando

Peça faltando se vê. Peça de mentira **passa em toda inspeção humana**: abre,
responde, confirma. E o problema não é ela — é o que se constrói em cima.

O passo seguinte natural era montar a fila de moderação das denúncias. Ela teria
ficado pronta, bonita, e vazia para sempre, porque a tabela que ela lê nunca
recebeu nada. Duas semanas depois alguém investiga "por que ninguém denuncia
neste site" olhando para o funil errado.

## A regra

Enquanto a peça não escreve, ela **diz que não escreve**. Um estado desabilitado
com "em breve", um `TODO` visível, qualquer coisa — menos a confirmação de
sucesso, que é justamente a afirmação que ela não pode sustentar.

E a confirmação, quando existir, é consequência da **resposta**, nunca do
clique. Fechada no clique, a torrada aparece igual quando a escrita falha, e a
peça volta a mentir — agora só às vezes, que é pior de achar.

## Como varrer os que já existem

Procure por confirmação sem `await`: torrada, modal que fecha, mensagem de
sucesso disparada no mesmo `onClick` que deveria ter enviado algo. Se entre o
clique e a confirmação não houve uma ida ao servidor, a confirmação é decoração.

## Conexões
- Princípio: [[Contador que conta sucesso de promessa afirma que deu certo]] —
  a mesma falsidade: "não falhou" e "fez o trabalho" são frases diferentes, e
  aqui nem tentativa houve.
- Irmã: [[Todo estado da tela tem visual]]
- Irmã: [[Catálogo de componentes é contrato vivo, não documentação]]
- Visto em: [[Privello]]
- Mapa: [[Design]]
