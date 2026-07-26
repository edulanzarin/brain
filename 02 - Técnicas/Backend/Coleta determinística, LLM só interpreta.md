---
tags: [tipo/atomica, camada/tecnica, dev/backend]
criado: 2026-07-26
---

# Coleta determinística, LLM só interpreta

> Ao usar um LLM sobre dados de um sistema (relatório, laudo, BI), o código coleta e organiza TUDO de forma determinística; o LLM recebe os dados prontos e só **interpreta**. Ele nunca busca o dado nem faz a conta que o código faz melhor.

O erro tentador é pedir pro LLM "analisar o banco" — deixá-lo montar query, somar, cruzar. Ele erra conta, alucina coluna e o custo/latência explode. A divisão certa:

- **Backend (determinístico):** puxa os dados (SQL), agrega, calcula o que é aritmética pura (variações, totais, séries) e **seleciona/compacta** o que mandar (não joga a base inteira — escolhe as linhas que importam, corta token).
- **LLM (interpretação):** lê os dados prontos e produz o julgamento — pontos fortes/fracos, alertas, recomendações, leitura de tendência. É onde ele agrega valor de verdade.

Onde a fronteira fica cinza, decida por **confiabilidade**: se o resultado varia com a estrutura do dado (ex.: grupos de conta que mudam de um plano contábil pro outro), o LLM lendo os rótulos é mais robusto que uma regra fixa no código; se é conta exata que o código faz igual sempre, é do código. O critério é "quem erra menos aqui", não "quem é mais esperto".

Ganhos concretos: custo baixo (manda só o essencial), resultado auditável (o dado que entrou é determinístico e reproduzível), e o LLM focado no que só ele faz.

Complementos práticos: **saída estruturada** (JSON schema) pra resposta cair direto na UI/PDF sem parsing frágil; **prompt caching** no system prompt fixo (as instruções não mudam entre execuções); e a chamada externa com timeout/erro tratado — ver [[Chamada externa tem timeout e erro tratado]].

- Princípio: [[Chamada externa tem timeout e erro tratado]]
- Visto em: [[Navetech Hub]] (Análise de Balancete: saldos vêm do Questor por SQL, o Claude escreve o laudo)
