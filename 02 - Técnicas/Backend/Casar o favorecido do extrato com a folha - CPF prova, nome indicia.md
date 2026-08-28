---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-28
---

# Casar o favorecido do extrato com a folha - CPF prova, nome indicia

> Comissão paga por fora da folha cai no extrato como um PIX qualquer, e a conta contábil
> depende de algo que só a folha sabe: o favorecido é funcionário? O extrato dá um nome
> sujo de jargão bancário e, com sorte, um CPF mascarado. Os dois casam com a folha — mas
> com forças diferentes, e tratá-los igual é o que transforma um palpite em lançamento.

## O que o extrato entrega

A descrição vem em formato imprevisível e carrega três coisas misturadas: jargão do banco
("PIX ENVIADO", "TED", "PAGTO", "LIQUIDACAO"), o nome do favorecido (às vezes **truncado**
— "JOAO CARLOS DE OLIVE") e, em muito PIX, o **CPF mascarado**: `***.456.789-**`.

O mascarado é melhor do que parece. O que o banco esconde são as três primeiras posições e
as duas últimas — justamente as menos discriminantes (as últimas são dígito verificador).
O que ele **mostra** é o miolo, e seis dígitos de miolo separam qualquer um numa base de
19 mil pessoas.

## As duas forças

| sinal | força | por quê |
|---|---|---|
| CPF (cheio ou mascarado) | **prova** | seis dígitos batendo posição a posição não colidem na prática |
| nome completo idêntico | indício forte | homônimo existe e é comum em base de folha |
| nome truncado ou parcial | indício fraco | "OLIVE" casa OLIVEIRA e OLIVEIRO |

A regra que sai daí: **o casamento por documento pode afirmar; o casamento por nome
acompanha a decisão sem tomá-la.** Nome que casa vira contexto na tela ("Funcionário
desde 03/2021, confira"), nunca a conta escolhida sozinha.

## Como casar sem produzir gente errada

```
1. normalizar: sem acento, caixa alta, só letras e dígitos
2. tirar o jargão do banco por lista (PIX, TED, PAGTO, TARIFA, LIQUIDACAO…)
3. CPF: achar padrões de 11 posições (dígito ou curinga), casar só as conhecidas
4. nome: exigir PRIMEIRO NOME + ao menos um sobrenome; token único nunca casa
5. cobertura: token exato vale 1, prefixo de 3+ letras vale 0,5 (o truncado do banco)
6. empate entre pessoas DIFERENTES → devolve o aviso de homônimo, não a escolha
```

Três detalhes que só aparecem implementando:

- **A lista de ruído é o que salva o passo 4.** Sem tirar "PAGTO" e "TARIFA" da descrição,
  eles viram tokens e casam com sobrenome de alguém.
- **Espaço fora da classe do CPF.** Uma regex tolerante a espaço junta "123 456 789 00" de
  campos vizinhos e inventa um CPF que casa com um inocente.
- **Multi-vínculo não é homônimo.** A mesma pessoa com dois contratos casa duas vezes e
  isso é normal; homônimo é CPF (ou nome, na falta dele) **diferente** empatando.

## Onde buscar, e a armadilha do escopo

Buscar só na empresa do extrato erra o caso mais confuso: em grupo econômico, a pessoa
presta serviço para uma empresa e é registrada na outra. Então o casamento cruza as
empresas — **mas só as que a sessão já enxerga**, senão a ferramenta vira um jeito de
descobrir funcionário de cliente que não é seu ([[Escopo de dado se clampa no servidor, num funil só]]).

Para o casamento não vale a view `funcionario` do Questor, que resolve vigências com um
join por tabela: nome, CPF e datas saem direto de `funccontrato` + `funcpessoa`. São ~21
mil vínculos na base inteira — cabem em memória, viram índice por token e casam a prévia
inteira do extrato numa consulta só, mais rápido que um ILIKE por linha.

## O que mais vale lembrar

- **Desligado tem que aparecer com a data.** Acerto de comissão depois da rescisão é caso
  comum, e "não está na folha hoje" responde a pergunta errada.
- O selo informa e não classifica **de propósito**: se a pessoa é funcionário, a comissão
  provavelmente deveria passar pela folha como rubrica (incide encargo), o que é decisão
  do contábil e não do casamento.

## Conexões
- Princípio: [[Casar dado do mundo real é por classe de equivalência, não por igualdade]]
- Irmã: [[Casar telefone brasileiro tolerando o nono dígito]] ·
  [[Estimativa fraca informa, número verificado ordena]] ·
  [[Ler extrato bancário em PDF]]
- Depende de: [[Módulo de folha e eSocial do Questor]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Backend]]

<!-- Candidato a princípio quando aparecer um terceiro caso: "evidência que não
     identifica sozinha acompanha a decisão, não a toma". Hoje tem dois (este e
     [[Estimativa fraca informa, número verificado ordena]]) e ambos em contexto de
     número/identidade; esperar um caso fora disso antes de promover. -->
