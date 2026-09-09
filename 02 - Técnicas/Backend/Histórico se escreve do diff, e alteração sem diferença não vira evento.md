---
tags: [tipo/atomica, camada/padrao, dev/backend]
criado: 2026-09-09
---

# Histórico se escreve do diff, e alteração sem diferença não vira evento

> "Dados atualizados" prova que alguém clicou em salvar. Não é isso que quem lê
> a linha do tempo foi buscar — ele quer saber qual campo mudou e para quanto.

## O problema

A linha do tempo do item nasce fácil: no fim do `PUT`, grava um evento
`updated`. O resultado, semanas depois, é uma pilha de linhas idênticas — mesma
frase, autores diferentes, horários diferentes. O registro existe, a informação
não. Pior: como salvar sem mexer em nada também grava, metade da pilha é ruído
puro, e o histórico que deveria explicar uma cobrança vira uma lista de cliques.

Quem abre esse painel tem uma pergunta concreta ("o honorário mudou quando?",
"quem trocou o vencimento?") e a resposta está no banco — só não está escrita.

## A solução

O evento é escrito a partir da **comparação entre o antes e o depois**, no mesmo
request:

1. lê o estado atual antes do `update`, já com os rótulos resolvidos (nome do
   grupo, da empresa — id não se lê),
2. aplica o `update` e usa a linha devolvida como "depois",
3. compara campo a campo e monta uma linha por diferença real,
4. **se não sobrou nenhuma linha, não grava evento nenhum.**

```ts
const field = (label: string, a: string, b: string) => {
    if (a !== b) lines.push(`• ${label}: ${a || "—"} → ${b || "—"}`);
};
// ...
return lines.length ? lines.join("\n") : null;   // null = nada a registrar
```

O que sai é o que a pessoa foi buscar: `• Honorário mensal: R$ 800,00 → R$
950,00`.

## O que mais vale lembrar

- **Campo sensível entra como fato, não como valor**: senha vira "• Senha
  alterada". O histórico é lido por mais gente do que o campo.
- **O "antes" tem que ser lido antes do update, na mesma requisição.** Ler
  depois é ler o depois duas vezes, e o defeito não aparece em teste de tipo.
- **Compare o rótulo, não o id.** `groupId` mudou de `9f2…` para `3ac…` não diz
  nada; "Grupo: Rede Sul → Rede Norte" diz.
- **O mesmo formato serve para o cadastro**: no `created`, em vez de comparar,
  liste só o que já entrou preenchido. Mantém uma leitura só na linha do tempo.
- Custa uma leitura a mais por edição. É barato perto de um histórico que não
  responde a pergunta nenhuma.

## Conexões
- Princípio: [[Nota carrega só o que a pessoa não sabe]] — "dados atualizados" é
  eco do próprio clique; o que a pessoa não sabe é qual campo e quanto era antes.
- Irmã: [[Criar e editar passam pelo mesmo funil de resolução]] ·
  [[A trilha de auditoria já é o placar de atividade, não crie tabela de métrica à parte]]
- Visto em: [[Cofre Digital]]
- Mapa: [[Backend]]
