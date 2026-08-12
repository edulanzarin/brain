---
tags: [tipo/atomica, camada/padrao, dev/backend, sql]
criado: 2026-08-12
---

# Posse numa permissão binária é duas seções e recorte por linha

> "O analista vê só os SEUS, o gestor vê TODOS" não é um nível de permissão — é
> uma questão de POSSE. Onde a permissão é binária (a seção é acessível ou não,
> sem view/edit), resolve-se com **duas seções** e um **recorte por linha** no
> servidor, não com um grau a mais na mesma seção.

## O problema

A tentação é dar à seção dois níveis — "ver" (os meus) e "editar/gerir" (todos).
Mas quando a doutrina do sistema é permissão **binária** ([[Permissão composta
por papéis somados, não exceção por usuário]]: acessa ou não), esse "meio acesso"
não existe. Forçar um view/edit ali reintroduz o grau que o modelo removeu de
propósito.

## A solução

Duas coisas somadas:

1. **Duas seções.** Uma do dono (preenche e vê os seus) e uma de gestão (vê
   todos). O analista ganha a primeira; o gestor ganha as duas. É o corolário de
   "restringir o que se faz = separar em outra seção, não conceder".
2. **Recorte por linha no servidor.** O gate de rota só diz "entra ou não";
   quem filtra o DADO é o handler, pela sessão: sem a seção de gestão, a consulta
   é `where autor = eu`; com ela, é todos. A leitura de um item por id re-checa o
   dono (o gate de seção não sabe de quem é a linha) — mesma cautela de
   [[Drill-down por id foge do funil de escopo e precisa de gate próprio]]. E vale
   nas duas pontas: a rota **e** a página de servidor barram o não-dono, senão o
   link direto vaza.

Assim o dono é uma propriedade da LINHA (uma coluna `autor`), não um privilégio
concedido — e ninguém vê o alheio trocando a URL.

## Conexões
- Princípio: [[Permissão se valida no servidor, não na interface]]
- Irmã: [[Permissão composta por papéis somados, não exceção por usuário]] · [[Escopo de dado se clampa no servidor, num funil só]] · [[Drill-down por id foge do funil de escopo e precisa de gate próprio]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Backend]]
