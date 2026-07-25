---
tags: [tipo/atomica, camada/padrao, seguranca, dev/backend]
criado: 2026-07-25
---

# Formulário público por token opaco fica fora do gate de sessão

> Quando alguém SEM conta precisa responder algo (um supervisor externo, um
> fornecedor, um cliente), o **token opaco no URL é a credencial**. A página e a
> submissão ficam **fora** do gate de sessão — o middleware não redireciona para
> o login e a rota não exige sessão. Quem autoriza é o próprio token: aleatório,
> difícil de adivinhar, ligado a exatamente um recurso.

O erro natural é reusar o mesmo gate de todo o resto (redirect otimista pro
login + wrapper de auth na API). Isso barra justo quem o link deveria alcançar:
a pessoa não tem — e não deve ter — login. A solução é abrir uma exceção
explícita e estreita:

- **Middleware/proxy**: excluir o caminho do formulário do matcher que manda pro
  login (ex.: `(?!api|login|experiencia|...)`).
- **Rota de submissão**: NÃO passar pelo wrapper de auth (que exige sessão). É um
  handler comum; a validação é o token.
- **Autorização = o token**: entropia alta (`randomBytes(24).toString("base64url")`),
  resolve o recurso, e é **consumível/idempotente** — respondido uma vez, recusa
  o resto (`on conflict do nothing` + status). Assim o link vazado não vira
  reenvio infinito nem sobrescreve resposta.

Cuidados que continuam valendo: sanitizar a entrada (o mundo manda o corpo),
vazar só o necessário pela existência do token (nome do avaliado, não a base
inteira), e manter TODO o resto do app gateado — a exceção é cirúrgica, um
caminho só.

É o espelho de [[Servir anexo por rota com checagem de permissão]]: lá a rota
checa a sessão para liberar o anexo; aqui o token SUBSTITUI a sessão porque não
há usuário logado. Nos dois a doutrina é a mesma —
[[Permissão se valida no servidor, não na interface]] — só muda quem é a
credencial.

Visto em: [[Navetech Hub]] (formulário de avaliação de experiência do RH: o
supervisor responde por link, sem logar).
