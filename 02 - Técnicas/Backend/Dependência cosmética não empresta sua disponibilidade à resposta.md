---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-30
---

# Dependência cosmética não empresta sua disponibilidade à resposta

> Uma tela cuja fonte de verdade é o SEU banco costuma tocar o sistema externo por um detalhe de apresentação — o nome de uma empresa, a foto de um usuário, o rótulo de um código. Se essa chamada for tratada como as outras, a tela herda a disponibilidade do externo e cai junto com ele, por um enfeite. Isole a chamada cosmética num `try` com fallback legível, e diga na tela que o nome está degradado.

## O problema

Numa seção com cinco abas que leem o ERP, a sexta lê o banco do próprio app — é
a única que deveria responder com o ERP fora do ar. Só que ela junta o nome da
empresa para não mostrar código cru, e essa junção a derruba igual às outras. O
resultado é pior que não ter a aba: quem abre no dia em que o ERP caiu conclui
que o sistema inteiro está fora.

A armadilha é que o código parece certo. O `await` do nome está no mesmo bloco
dos outros, o erro sobe pelo mesmo caminho, e a diferença de **importância**
entre "o dado" e "o rótulo do dado" não aparece em lugar nenhum.

## A solução

Separar por criticidade, no ponto da chamada:

```ts
async function nomearEmpresas(codigos: number[]) {
  const fallback = { nome: (c: number) => `Empresa ${c}`, resolvidos: false };
  if (codigos.length === 0) return { ...fallback, resolvidos: true };
  try {
    const linhas = await query(/* ... */);
    const mapa = new Map(linhas.map((e) => [e.codigo, e.nome]));
    return { nome: (c) => mapa.get(c) || `Empresa ${c}`, resolvidos: true };
  } catch (err) {
    console.error("[prod-app] externo fora — empresas sem nome:", err);
    return fallback;   // a resposta sai; só o rótulo degrada
  }
}
```

Três detalhes que fazem a diferença:

1. **O fallback é legível**, não vazio: `Empresa 1200` ainda identifica a linha,
   ordena e exporta. Rótulo nulo transformaria degradação em bug.
2. **A resposta carrega o sinal** (`resolvidos`), e a tela avisa no rodapé. Sem
   isso o usuário conclui que o cadastro sumiu — troca uma falha visível por uma
   invisível, que é pior.
3. **Nada a resolver conta como sucesso**, não como falha: lista vazia devolve
   `resolvidos: true`. Senão a tela avisa de uma queda que não houve.

## O que mais vale lembrar

- O teste da regra é a pergunta: *se isto falhar, a resposta ainda responde à
  pergunta que foi feita?* Nome de empresa: sim. Valor do lançamento: não — esse
  é crítico e deve derrubar.
- Vale além de nome: avatar, descrição de código, rótulo de enum vindo de
  cadastro, geocodificação de endereço. Tudo que enfeita o identificador.
- O `catch` que engole precisa **logar**, ou a degradação vira mistério. Best-
  effort não é silêncio.

## Conexões
- Princípio: [[Ausência de leitura cai no valor que dispara a ação]] — aqui a ausência cai no rótulo neutro, que é o valor que não muda nenhuma decisão
- Irmã: [[Chamada externa tem timeout e erro tratado]]
- Visto em: [[Navetech Hub]] — aba No Nexo, a única da seção que responde com o Questor fora
- Mapa: [[Backend]]
