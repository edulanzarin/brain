---
tags: [tipo/atomica, camada/padrao, dev/backend, seguranca, armadilha]
criado: 2026-08-24
---

# Recorte pequeno em pesquisa anônima identifica, então o painel se recusa a mostrar

> Guardar a resposta sem nome não basta para ser anônimo: um painel com filtro por setor e por tempo de casa reconstrói a pessoa em dois cliques. O anonimato se mantém no que o painel MOSTRA, não só no que o banco guarda.

## O problema

Pesquisa de clima anônima: nenhuma coluna de identidade, nenhum vínculo com pessoa — a promessa está cumprida no schema. Aí o painel ganha recorte por segmento, que é justamente o que o RH precisa ("como o Contábil respondeu?"), e a promessa se desfaz sem ninguém escrever uma linha maliciosa: setor Contábil **e** menos de 6 meses de casa costuma ser uma pessoa só, e a resposta escrita dela aparece na tela com nome e sobrenome implícitos.

Quem respondeu confiando no anonimato não distingue "o banco não guarda quem" de "o painel não deixa chegar em quem". Para ela, é a mesma promessa.

## A solução

Um piso de respostas por recorte, checado antes de renderar qualquer coisa:

```ts
export const MIN_ANONIMATO = 3;
const escondido = anonimo && segmento != null && filtradas.length < MIN_ANONIMATO;
```

Abaixo do piso o painel troca todo o resultado por uma explicação — "são 2, abaixo de 3; escolha um recorte mais amplo" —, e a **exportação cai junto**: recorte que não se mostra também não vira planilha. Sem o piso valendo para os dois caminhos, o botão de exportar é a porta dos fundos.

## O que mais vale lembrar

- O piso só vale **com recorte ativo**: o total da rodada não identifica ninguém, mesmo com 2 respostas.
- É trava de estrutura, não de aviso: recomendar "não filtre demais" deixa o invariante nas mãos de quem está com pressa.
- O mesmo raciocínio vale para qualquer agregado sobre gente — ranking por equipe pequena, média salarial por cargo com um ocupante.
- Combinação de filtros derruba o n rápido; se um dia houver dois recortes ao mesmo tempo, o piso continua sendo sobre o resultado FINAL do cruzamento.

## Conexões
- Princípio: [[Anonimato se perde na saída, não só na entrada]]
- Depende de: [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[Canal anônimo não guarda quem, e o retorno é um segredo do denunciante]]
- Irmã: [[Formulário montado pelo usuário — a definição no banco dirige renderer e validação]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Backend]]
