---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-24
---

# Adquirir o recurso exclusivo é uma ação, usá-lo é outra

> Quando o sistema depende de um recurso que só existe em uma cópia — uma sessão
> que o servidor concede a um cliente por vez, um lock, uma conexão que dá direito
> exclusivo — é natural escrever "ligar" como a mesma operação que a primeira
> tarefa que se quer fazer com ele. O botão fica sendo "começar a caçar", e ele
> faz duas coisas: toma a sessão e entra na caçada. Funciona, até aparecer a
> segunda tarefa.

## O que quebra

No [[piwdex2]] o robô toma a sessão de jogo (o WebSocket **é** a sessão, e o jogo
aceita uma por conta) e faz quatro coisas com ela: caça, vende, repõe consumível
e conversa no chat. Ligar exigia escolher uma caçada, e disso vinham três
defeitos que pareciam não ter relação entre si:

- **Quem quer só vender era obrigado a caçar.** A tarefa escolhida virou
  pré-requisito das outras três.
- **Trocar de caçada passava por desligar.** No intervalo, o servidor devolvia a
  sessão para a aba do navegador — o recurso escapava por causa de uma troca que
  nem precisava tocá-lo.
- **A tela não conseguia explicar o estado.** "Parado" queria dizer duas coisas
  diferentes (sem sessão / com sessão e sem tarefa), e a diferença é justamente o
  que o usuário precisa saber.

## A separação

Duas ações, dois estados, e a tarefa nunca toca a aquisição:

```
POST /ligar    adquire e SEGURA. Não pede tarefa nenhuma.
POST /cacar    entra, troca ou sai da caçada, sobre o que já está de pé.
POST /parar    devolve o recurso.
```

- **O estado desejado vira dois campos**, não um: "quero segurando" e "qual
  tarefa". Um só obrigava a inventar uma tarefa para expressar a intenção de
  segurar.
- **Trocar de tarefa não passa pela aquisição.** É a propriedade que mais paga:
  sair de uma caçada e entrar na outra acontece pelo mesmo socket, e o recurso
  nunca fica sem dono.
- **Retomar depois do restart segura mesmo sem tarefa.** Exigir uma tarefa para
  religar deixaria de fora exatamente quem usa o sistema para as outras coisas.

## O que mais vale lembrar

- O teste que revela a confusão é uma pergunta: **"e se eu quiser fazer só a
  outra coisa?"** Se a resposta obriga a escolher a primeira, as duas ações estão
  coladas.
- Segurar recurso exclusivo é uma decisão com custo para terceiros — no caso, o
  dono da conta perde a própria aba. Isso é motivo a mais para a aquisição ser
  explícita e ter um estado próprio na tela, em vez de ser efeito colateral de
  outro botão.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[Estado desejado persistido religa o robô depois do restart]] ·
  [[Processo que guarda conexão viva não tolera deploy frequente, e o log não denuncia]]
- Visto em: [[piwdex2]]
- Mapa: [[Backend]]
