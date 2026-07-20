---
tags: [tipo/atomica, camada/principio, dev/backend]
criado: 2026-07-20
---

# Chamada externa tem timeout e erro tratado

> Tudo que sai do processo pode travar e pode falhar. Se não tem prazo pra
> desistir e caminho pro erro, não está pronto.

## A regra

Vale pra processo filho, requisição HTTP, query, leitura de arquivo, fila — qualquer
coisa cujo tempo de resposta eu não controlo:

- **Timeout sempre**, com encerramento de verdade. Em processo filho, `kill("SIGKILL")`
  depois do prazo; num fetch, `AbortController`. Timeout que só registra e continua
  esperando não é timeout.
- **Capturar o canal de erro**, não só o de sucesso. É no `stderr` que vem a causa real
  ("Incorrect password", "Permission denied"); sem ele a mensagem ao usuário vira um
  genérico inútil e a depuração começa do zero.
- **Prever a falha do meio**: o outro lado pode fechar a conexão antes de consumir tudo
  (`EPIPE`), responder 200 com corpo vazio, ou devolver HTML no lugar de JSON.
- **Traduzir o erro** antes de mostrar. O usuário precisa saber o que fazer, não ver o
  stack trace.

## Por que

A falha que dói não é a que estoura — é a que **fica pendurada**. Sem timeout não há
exceção, não há log e não há erro: a requisição simplesmente não volta, o usuário
recarrega, e cada tentativa segura mais uma conexão. Um arquivo corrompido derruba o
servidor inteiro sem nunca aparecer no monitoramento.

Foi assim ao mandar PDF por stdin: o processo filho não consumia a entrada, ninguém
reclamava, e a requisição só morria no timeout do servidor — quando existia um.

## Corolário

Se a chamada externa é lenta por natureza (assinatura, processamento, integração), a
resposta não é aumentar o timeout: é tornar a operação assíncrona e consultar o
resultado depois — ver [[Polling substitui webhook quando não há IP público]].

## Conexões
- Irmã: [[Verificar no build de produção, não só em dev]]
- Técnicas que aplicam: [[Armadilhas de child_process no Node]] · [[Ler extrato bancário em PDF]] · [[Polling substitui webhook quando não há IP público]]
- Mapa: [[Base]] · [[Backend]]
