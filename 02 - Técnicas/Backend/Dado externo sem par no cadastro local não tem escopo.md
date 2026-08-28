---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-28
---

# Dado externo sem par no cadastro local não tem escopo

> Ao trazer uma fonte externa para dentro, parte das linhas não vai casar com o
> seu cadastro. Essas linhas ficam **sem a chave pela qual o escopo recorta** — e
> a saída preguiçosa, tratar o vazio como "vale para todos", entrega dado de
> quem não devia ser visto.

## O problema

O recorte por escopo é uma condição sobre uma chave local (`empresa_id`,
`cliente_id`). A fonte externa traz identidade dela — um CNPJ, um e-mail, um
código de terceiro — e você resolve o par na sincronização. Nunca casa 100%: o
outro sistema cadastra o que o seu não tem, ou parte de outra granularidade (no
caso medido, filial como empresa própria, o que sozinho respondia por 179 de
1.200 pares perdidos até o casamento incluir todos os estabelecimentos, não só a
matriz).

O que sobra sem par é onde mora a armadilha. `chave is null` no filtro tem duas
leituras opostas:

- **"vale para todos"** — correta para um evento de sistema que não pertence a
  cliente nenhum;
- **"não sei de quem é"** — correta para um dado de cliente que não foi
  resolvido.

Escolher a primeira por reflexo (é o que se faz em trilha de auditoria) mostra a
carteira alheia a quem tem escopo restrito.

## A regra

Sem par, **restrinja a quem vê tudo**. A linha continua existindo, continua sendo
contada num indicador de saúde da integração ("N sem par"), mas só aparece para
quem já teria acesso a qualquer cliente. É o default que erra para o lado de
esconder, e esconder demais se percebe; vazar, não.

E torne o número visível: "10% sem par" é diagnóstico de integração, não detalhe.
Enquanto ele estiver na tela, alguém pode decidir melhorar o casamento — enquanto
estiver só no código, ninguém sabe que existe.

## Conexões
- Princípio: [[Permissão se valida no servidor, não na interface]]
- Irmã: [[Escopo de dado se clampa no servidor, num funil só]] · [[Sobre fonte read-only, o editável mora no seu banco chaveado pela identidade dela]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Backend]]
