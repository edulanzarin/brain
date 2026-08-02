---
tags: [tipo/atomica, camada/padrao, dev/backend, dados]
criado: 2026-08-01
---

# Entidade núcleo cresce por tabela satélite, não por coluna

> Mantenha a entidade central **mínima e estável** (só o que é sempre verdade). Todo
> atributo que pertence a um domínio específico entra numa **tabela nova que referencia**
> a núcleo — não como coluna adicionada nela. Assim o núcleo não muda quando o produto
> cresce; ele ganha satélites.

## Por quê

Colocar tudo na tabela central parece prático, mas cada feature nova vira uma alteração
de schema no coração do sistema — migração arriscada, campos que só valem pra metade das
linhas, e acoplamento entre módulos que não deviam se conhecer. Quando o atributo vive
numa tabela própria, a feature é **aditiva**: cria tabela, referencia a núcleo, pronto.
Some a feature? Dropa a tabela satélite, a núcleo nem sente.

## Como

- A núcleo guarda a identidade e o que é universal. Ex.: `Contact = { id, nome, telefone }`.
- Extensões de domínio referenciam por FK: `ContactCrm(contactId → Contact)`,
  `ContactEmpresa(contactId, empresaId)`, `ContactPreferencia(contactId, …)`.
- A UI/consulta compõe: lê a núcleo e faz `join`/`include` do satélite quando aquele
  módulo está ligado.
- O mesmo vale pra **histórico**: em vez de sobrescrever campos, um satélite de eventos
  (`*_evento` append-only) preserva a linha do tempo sem inchar a núcleo.

## Sinais de que você deveria ter feito isso

Coluna que é `null` na maioria das linhas; migração que toca a tabela mais central a cada
release; um módulo que "precisa" de um campo que só ele usa. Tudo isso é satélite querendo
nascer.

## Conexões
- Visto em: [[navetalks]] — Contact ficou mínimo de propósito; CRM (empresa, honorário,
  etiquetas) vai virar tabela satélite, sem alterar `Contact`.
- Relacionado: [[Formulário montado pelo usuário — a definição no banco dirige renderer e validação]]
- Mapa: [[Dados]]
