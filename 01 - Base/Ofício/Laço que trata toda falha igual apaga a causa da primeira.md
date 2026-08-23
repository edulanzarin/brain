---
tags: [tipo/atomica, camada/principio, armadilha]
criado: 2026-08-23
---

# Laço que trata toda falha igual apaga a causa da primeira

> Repetir uma operação até dar certo é barato de escrever e caro de diagnosticar.
> Quando o `catch` não distingue POR QUE falhou, a primeira tentativa — a única
> que carregava a causa real — é sobrescrita pelas seguintes, e o erro que chega
> na tela é o da última, que quase sempre é um sintoma do próprio laço.

## Os dois casos

**Um runner de migration** conectava ao Postgres com quinze tentativas espaçadas.
A saída era quinze linhas de "banco indisponível, aguardando..." e, no fim,
`Client has already been connected`. Nenhuma das duas era verdade: o banco estava
de pé, e o erro final era do próprio laço reusando um cliente que o `pg` marca
como queimado depois de um connect falho. A causa verdadeira aparecia uma vez só,
na primeira volta, e era **senha errada** — apagada pelas outras quatorze.

**Um robô que reconectava** a um serviço externo tratava 401, 403 e 429 como a
mesma coisa. Só que os três pedem o oposto: renovar o token, parar de vez e
esperar. Confundidos, o robô batia na porta de uma conta banida indefinidamente,
e o dono via só "não conectou".

A forma é a mesma nos dois: **o laço transforma um diagnóstico em silêncio.**

## O que fazer

- **Guarde a primeira falha, não a última.** Se o laço vai desistir, é a primeira
  que explica; as posteriores são consequência.
- **Classifique antes de repetir.** Falha transitória (rede, serviço subindo) se
  repete; falha estrutural (credencial errada, recusa, recurso inválido) não se
  resolve nunca insistindo, e insistir só atrasa a descoberta.
- **Recurso queimado não se reusa.** Cliente, socket e stream costumam ficar
  inválidos depois de um erro. Construa um novo a cada tentativa — senão o laço
  passa a medir a si mesmo.
- **Imprima o motivo em cada volta**, não só a contagem. "Tentativa 3/15" não
  informa nada; "tentativa 3/15: password authentication failed" encerra o
  assunto na terceira linha.

## Conexões
- Irmã: [[Chamada externa tem timeout e erro tratado]] ·
  [[Sonda que falhou não é sinal de que mudou]]
- Exemplo: [[Recusa não é falha: contra o não do servidor, insistir é ruído]] ·
  [[Retry que reusa o cliente queimado esconde o erro da primeira tentativa]]
- Mapa: [[Base]] · [[Backend]]
