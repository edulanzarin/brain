---
tags: [tipo/atomica, camada/padrao, dev/backend, dados]
criado: 2026-07-31
---

# Deixar o método da conferência visível quando o SQL não foi validado

> Quando uma conferência depende de uma **heurística** que você não pôde validar
> contra o dado real (que conta casa, que registro conta, que filtro isola), a
> tela não deve cuspir só o veredito — deve **mostrar a memória de cálculo**: as
> linhas que entraram em cada lado. Assim o humano que tem o dado valida o método,
> e um número errado se denuncia em vez de enganar.

## O atrito

Automação sobre banco de terceiro (ex.: o Questor, produção read-only) às vezes se
escreve sem acesso pra rodar a query — o schema vem de documentação, e a
identificação é heurística: "provisão é a conta cujo nome contém 'provis'", "a
folha de provisão é o `tipocalculo` 70/71". Se a tela mostra só "divergência: R$
1.500", ninguém sabe se a divergência é real ou se a heurística pegou conta demais
/ de menos. Um número solto de uma regra não-validada tem a **mesma aparência** de
um número certo — e é aí que ele engana.

## O padrão

Ao lado do veredito, listar o que o compôs, dos dois lados:

- O "calculado" mostra **quais registros** somou (as folhas, com tipo e total).
- O "lançado" mostra **quais contas** entraram (número, descrição, valor).
- Um rodapé declara o método em uma frase ("calculado = X; lançado = Y").

Com isso, quem conhece a empresa bate o olho: "faltou a conta 2.1.05", "essa folha
não era provisão". O método vira **auditável na própria tela**, e a validação que
você não pôde fazer acontece no primeiro uso, por quem tem o dado.

Corolário: registre a heurística como **pendência explícita** (no código e na nota
do projeto) — "a validar contra empresa conhecida" —, não como fato consumado.

## Fronteira

Isto é para o que **não deu** pra validar. Onde a lógica foi conferida contra o
real (bateu ao centavo numa empresa conhecida), o veredito basta — expor a memória
de tudo vira ruído. A memória visível é o preço de embarcar uma heurística; some
quando ela deixa de ser heurística.

## Conexões
- Parente: [[Coleta determinística, LLM só interpreta]] (separar o que se calcula do que se interpreta)
- Parente no fechamento: [[Fluxo de fechamento é orquestração dos motores que já existem]]
- Doutrina de fundo: [[Questor - conexão read-only e regras]]
- Visto em: [[Navetech Hub]] — Contábil, seção Provisões: folha calculada × contábil lançado, com as folhas e contas visíveis porque a identificação (tipo 70/71, conta "provis") é heurística a validar no banco real.
- Mapa: [[Backend]]
