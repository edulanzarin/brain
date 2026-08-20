---
tags: [tipo/atomica, camada/padrao, infra, armadilha]
criado: 2026-08-19
---

# Processo que guarda conexão viva não tolera deploy frequente, e o log não denuncia

> Serviço que segura estado vivo em memória (WebSocket, sessão, worker de longa duração)
> morre inteiro a cada deploy. Com auto-deploy ligado, **cada push derruba todo mundo** —
> e o log de produção não acusa nada, porque um processo novo escreve "Ready" igualzinho
> ao que estava rodando há horas.

## O problema

No [[piwdex]] o robô segura um WebSocket por usuário, em memória. Uma noite nenhuma conta
conectava mais. Os logs, lidos de cima, pareciam saudáveis: `Ready` em todo ciclo, sem
stack trace, sem erro de aplicação.

O que estava acontecendo era deploy repetido — e **eu era parte da causa**, com seis
pushes na `main` durante uma sessão de trabalho, cada um disparando um deploy pelo
auto-deploy do GitHub.

O que fechou o diagnóstico foi comparar duas escalas de tempo que viviam em camadas
diferentes e nunca tinham sido postas lado a lado:

| Camada | Número |
|---|---|
| tempo que o container ficava de pé | ~13 s |
| `CONTESTED_MS`: tempo que a conexão precisa durar pra "vencer" a sessão | 25 s |

**13 < 25.** A conexão nunca sobrevivia o suficiente. Pior: cada queda antes da janela
contava um strike de "conta tomada", e o robô se pausava sozinho por um motivo inventado.
Nenhum dos dois números estava errado — errado era ninguém tê-los comparado.

## A solução

- **Sonda de uptime, não de "está no ar".** `GET /api/health` devolvendo `uptimeSeconds`
  responde a pergunta que o log não responde: abra duas vezes com minutos de intervalo e,
  se o número voltou pra perto de zero, o processo morreu no meio. Healthcheck que só
  responde 200 não distingue "vivo há 3 horas" de "nasceu agora".
- **Compare a vida do processo com a menor janela interna que ele precisa cumprir.**
  Toda constante de tempo do app (janela de handshake, backoff, TTL, lock) tem um
  requisito implícito de sobrevivência. Se o processo vive menos que a menor delas,
  o sistema entra num modo de falha que não se parece com "container reiniciando".
- **Auto-deploy e estado vivo são um par ruim.** Ou o deploy vira ato deliberado (mão no
  botão, janela combinada), ou o estado precisa ser reconstruível a ponto de o corte não
  importar — e reconstruir só resolve se o processo novo tiver tempo de terminar.
- **Deploy com sobreposição faz DOIS processos disputarem o mesmo recurso externo.** Com
  o antigo e o novo vivos ao mesmo tempo, os dois religam as mesmas sessões e brigam pela
  mesma conexão do outro lado, que aceita uma só. O sintoma sai como "fui chutado".

## O que mais vale lembrar

Ao diagnosticar por log, separe o que é **rotina barulhenta** do que é sinal: o Postgres
escreve `checkpoint` no stderr a cada 5 minutos e o Railway pinta tudo como `[error]`.
Duas telas de vermelho ali eram ruído, e o sinal de verdade estava numa linha discreta —
o pre-deploy rodando de novo, que só acontece em deploy, nunca em restart.

## Conexões
- Princípio: [[Ambiente de dev sobe igual ao de produção]]
- Irmã: [[Estado desejado persistido religa o robô depois do restart]] (a mitigação — que
  só funciona se o processo viver o bastante pra concluir a religação) ·
  [[Railway não roda compose, cada serviço vira uma peça da plataforma]]
- Visto em: [[piwdex]]
- Mapa: [[Infra]]
