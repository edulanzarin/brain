---
tags: [tipo/atomica, camada/padrao, design, dev/frontend]
criado: 2026-07-22
---

# Seletor cria e gerencia os próprios itens

> Um combobox de busca que também cria item novo ao digitar e renomeia/exclui os
> existentes no próprio painel — dispensa uma tela de CRUD para a entidade auxiliar.

## O problema

O seletor de uma entidade auxiliar (grupo, etiqueta) normalmente só escolhe entre o
que já existe. Criar, renomear ou excluir exige outra tela. Isso empurra o usuário
pra fora do fluxo justo na hora em que ele precisa do item —
[[Entidade auxiliar se cria no ponto de uso, não em tela própria]].

## A solução

Estender o combobox de busca (o mesmo que já filtra listas grandes) com dois modos,
ligados por props opcionais — sem eles, ele continua um seletor comum:

- **Criar**: quando o texto digitado não casa (comparação sem acento/caixa) com
  nenhum item, aparece uma linha "Criar «texto»". `onCreate(nome)` faz o POST,
  devolve o `value` criado e o seletor já o seleciona.
- **Gerenciar**: itens marcados `manageable` ganham lápis/lixeira no hover, dentro do
  painel. Renomear troca o rótulo por um input inline; excluir pede confirmação na
  própria linha. `onRename(value, nome)` e `onDelete(value)`.

```tsx
<Combobox
  options={groupOptions}          // itens com manageable: true nos de verdade
  value={group} onChange={setGroup}
  onCreate={criarGrupo}           // (nome) => Promise<idNovo>
  onRename={renomearGrupo}        // (id, nome) => Promise<void>
  onDelete={excluirGrupo}         // (id) => Promise<void>
/>
```

## O que mais vale lembrar

- **O painel é portal no `body`** (senão o `overflow` do modal corta a lista) —
  ver [[Portal condicional dispensa o flag de montagem]].
- **Excluir o item selecionado zera a seleção** (`onChange("")`), senão o filtro
  aponta pra um id que não existe mais.
- **O nome do auxiliar costuma vir embutido** na entidade principal já carregada
  (a empresa traz `group.name`). Depois de renomear/excluir, recarregue a lista
  principal, ou os rótulos antigos ficam na tela até o próximo fetch.
- **Permissão**: no formulário de certificado o campo de grupo só assina o grupo à
  empresa dona (nunca desvincula) e exige permissão de *empresas*, mesmo o cert
  sendo guardado por quem edita certificados. Vazio = não mexe.

## Conexões
- Princípio: [[Entidade auxiliar se cria no ponto de uso, não em tela própria]]
- Irmã: [[Controles de filtro do dashboard]]
- Visto em: [[Cofre Digital]]
- Mapa: [[Design]]
