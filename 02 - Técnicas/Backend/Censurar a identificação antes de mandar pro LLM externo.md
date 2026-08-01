---
tags: [tipo/atomica, camada/tecnica, dev/backend, seguranca]
criado: 2026-08-01
---

# Censurar a identificação antes de mandar pro LLM externo

> Quando o laudo/análise vai pra um LLM de terceiro, nome e CNPJ do cliente saem antes do envio; a IA recebe um marcador, e o nome real é reposto só na volta. A identificação nunca cruza a fronteira.

## O problema

O dado é de cliente da contabilidade — CNPJ, faturamento, estrutura patrimonial. Mandar isso pra um LLM externo (ainda mais em tier grátis, que pode treinar com o input) é sigilo profissional e LGPD. Mas a prosa do laudo depende do LLM. Quero os dois: a redação da IA sem entregar quem é a empresa.

## A solução

Censura em duas camadas, defesa em profundidade:

- **Na origem**: o payload já é montado com um marcador (`[EMPRESA]`) no lugar do nome, e o CNPJ nem entra. A identificação não é adicionada pra depois ser removida — ela nunca é escrita no que vai.
- **Varredura por cima** (scrub) antes de enviar: uma passada mascara qualquer nome/CNPJ/CPF que tenha escapado por outra via (descrição de conta, observação). Rede de segurança pro caso de o primeiro passo não cobrir tudo.

Duas armadilhas:

- **A regex de documento não pode comer número de verdade.** Valor em R$ vem agrupado em milhar (`R$ 12.345.678`); casar isso como CNPJ corromperia o dado que a IA precisa ler. A regex casa CNPJ formatado ou 14 dígitos crus, não grupos de milhar.
- **Repor na volta, do lado de cá.** O nome real volta no texto pronto por um `split/join` do marcador — depois que a resposta chegou, no servidor. O terceiro nunca viu; o usuário lê o laudo com o nome certo.

```ts
// origem: monta já com o marcador; CNPJ fora
`EMPRESA: [EMPRESA]`
// scrub antes de enviar
texto.replace(new RegExp(escapar(nome), "gi"), "[EMPRESA]")
     .replace(/\b\d{2}\.\d{3}\.\d{3}\/\d{4}-\d{2}\b|\b\d{14}\b/g, "[CNPJ]")
// volta: repõe o nome real
laudo.split("[EMPRESA]").join(nome)
```

## O que mais vale lembrar

A fronteira do terceiro é onde o teu controle acaba: o que sai não volta atrás. Então o que cruza é o mínimo necessário e sem quem-é. É o outro lado da mesma moeda da irmã [[Coleta determinística, LLM só interpreta]] — ela corta o volume (manda só o essencial, não a base inteira), esta corta a identificação.

## Conexões
- Princípio: folha isolada por ora — candidata a promover "pro terceiro vai só o mínimo, sem identificação" quando aparecer o 2º caso
- Irmã: [[Coleta determinística, LLM só interpreta]]
- Visto em: [[Navetech Hub]] (laudo de balancete: nome/CNPJ censurados antes de ir pra Groq)
- Mapa: [[Backend]]
