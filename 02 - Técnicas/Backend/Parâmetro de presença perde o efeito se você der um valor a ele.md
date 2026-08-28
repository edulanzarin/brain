---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-28
---

# Parâmetro de presença perde o efeito se você der um valor a ele

> Algumas APIs tratam certos parâmetros como BANDEIRA: valem por estarem na
> query (`?config`), não pelo que carregam. Escrever `?config=1` não é uma forma
> equivalente — para essas APIs é outro parâmetro, e o efeito some. Sem erro:
> a resposta volta vazia, como se não houvesse dado.

## Como se cai nisso

Testando à mão você digita `?config` — é o que a documentação mostra e o que dá
menos trabalho. Funciona, e você segue.

No código, ninguém concatena query string à mão: usa o construtor da linguagem
(`URLSearchParams`, `http.params`, `params={}`), e **todos eles escrevem
`chave=valor`**. Passar `config: "1"` parece a tradução óbvia de uma flag. O
resultado é uma requisição que só se parece com a que você testou.

O sintoma é cruel porque não aponta para a causa: a API respondeu 204, a lista
veio vazia, e a leitura natural é "não tem dado" ou "o filtro está estreito
demais" — nunca "meu parâmetro está malformado". Medido em ago/2026 na API do
Acessórias: `deliveries?...&config=1` devolve **HTTP 204, corpo vazio**;
`deliveries?...&config` devolve **200 com a lista inteira**. Duas varreduras de
meia hora colheram zero antes de alguém desconfiar da própria URL.

## A regra

**A requisição que você testou e a que o código emite têm que ser a mesma
string.** Quando o teste manual funciona e o código não, compare as duas URLs
inteiras antes de investigar qualquer outra coisa — a diferença costuma estar na
serialização, que é o pedaço que ninguém olha porque "é só a biblioteca".

Na prática: o cliente HTTP precisa de um caminho para emitir bandeira NUA, fora
do construtor de parâmetros. E o teste que prova a correção deve montar a URL
**pelo mesmo código do cliente**, não por um curl paralelo — senão você prova que
o curl funciona, que nunca esteve em dúvida.

## Conexões
- Princípio: [[Contador que conta sucesso de promessa afirma que deu certo]]
- Irmã: [[Campo cujo nome você não sabe se lê do payload, nunca se chuta]] · [[Dado externo sem par no cadastro local não tem escopo]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Backend]]
