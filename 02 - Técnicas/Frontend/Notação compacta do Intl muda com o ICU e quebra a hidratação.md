---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-09-24
---

# Notação compacta do Intl muda com o ICU e quebra a hidratação

> `Intl.NumberFormat(..., { notation: "compact" })` depende da versão do ICU de
> quem executa. O Node escreveu "R$ 412,0 mil" e o navegador "R$ 412 mil" para o
> mesmo número. Em componente renderizado no servidor, essa diferença de texto é
> um erro de hidratação do React (418), sem nenhuma pista de onde está.

## O que aconteceu

O catálogo do NaveX disparava o erro 418 só no build de produção. Formatação de
moeda, número inteiro, data e hora saía idêntica nos dois motores (conferido
lado a lado, com os dois no mesmo fuso: no container, em UTC, a hora diverge,
ver [[Hora formatada no servidor sai no fuso do container e quebra a hidratação]]); a compacta com `maximumFractionDigits: 1` não: o Node mantém o
",0" do número redondo, o Chromium corta. Com valor quebrado (84.210) as duas
dão "84,2 mil", e é por isso que o teste óbvio passa.

## Como achar

A mensagem minificada não diz o nó. O caminho que funcionou:

1. dividir a página (um parâmetro temporário que renderiza uma família por vez)
   até isolar o bloco;
2. tirar o texto do bloco duas vezes pelo mesmo `innerText`, com JavaScript
   desligado no navegador headless (o que o servidor mandou) e ligado (o que o
   React desenhou), removendo os gráficos, que só existem no cliente;
3. comparar os dois sem espaço e olhar a primeira divergência.

## A correção

Montar a forma compacta à mão: dividir pela unidade, formatar com o
`NumberFormat` padrão (que é estável entre motores) e acrescentar "mil", "mi" ou
"bi". Cuidar do arredondamento que chega a mil ("999,96 mil" sobe para "1 mi").
Um teste unitário fixa o texto esperado, com o espaço que não quebra.

## O que mais vale lembrar

- Vale para toda opção do Intl que dependa de dado de localização mais novo:
  `compact`, `unitDisplay`, nomes de moeda por extenso. O que é antigo e estável
  (separador de milhar, vírgula, `R$`) coincide.
- Página estática gerada no build tem o mesmo risco com outra causa: rótulo
  relativo a hoje ("Mês passado") congela no dia do deploy.

## Conexões
- Princípio: [[Verificar no build de produção, não só em dev]]
- Irmã: [[Ponto decimal em interface pt-BR afirma outro número]]
- Visto em: [[NaveX]]
- Mapa: [[Frontend]]
