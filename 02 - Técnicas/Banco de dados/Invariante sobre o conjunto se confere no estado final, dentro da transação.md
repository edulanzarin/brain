---
tags: [tipo/atomica, camada/padrao, dev/backend, sql]
criado: 2026-09-25
---

# Invariante sobre o conjunto se confere no estado final, dentro da transação

> "Sempre sobra um administrador ativo" não se garante vigiando cada caminho que pode quebrar a regra. Faz-se a escrita, confere-se o estado resultante na mesma transação e, se a regra caiu, desfaz-se tudo.

## O problema

O nexo2 só barrava o administrador de excluir a si mesmo. Os outros caminhos para ficar sem ninguém na Administração continuavam abertos:
- desmarcar "acesso total" do único cargo admin;
- apagar esse cargo;
- tirar o cargo do usuário;
- desativar o outro administrador.

Cada caminho pedia uma checagem própria, com a sua conta de "quem mais é admin", e o próximo caminho a nascer esqueceria a dele.

## A solução

Uma função que olha o estado, chamada no fim de toda escrita que pode afetá-lo, ainda dentro da transação:

```sql
select exists (
  select 1 from usuario u
    join usuario_cargo uc on uc.usuario_id = u.id
    join cargo c on c.id = uc.cargo_id
   where u.ativo and c.admin
) as ok
```

Se voltar falso, a função lança o erro com a frase para a tela e a transação faz rollback. A regra não precisa saber qual caminho a quebrou: ela vê o mundo depois da mudança, que é o único lugar onde a resposta é certa.

## O que mais vale lembrar

- **O que é do gesto continua no gesto.** "Não se exclua" e "não se desative" são sobre quem clica, e a mensagem própria vale mais que um "ninguém mais seria admin" genérico. A invariante é a rede de baixo, não a única trava.
- **Concorrência:** em `read committed`, dois administradores removendo um ao outro ao mesmo tempo passam os dois, porque cada um ainda vê o outro. Para fechar isso, trave as linhas lidas (`for update` nos cargos admin) ou rode a transação em `serializable`. Com um punhado de administradores e escrita rara, o risco foi aceito.
- Conferir no fim só vale se todas as escritas passam pelo mesmo funil de domínio. Um `update` solto numa rota fora dele fura a regra sem ninguém ver. A versão inteiramente estrutural é a mesma consulta num `constraint trigger ... deferrable initially deferred`, que o banco roda no commit de qualquer escrita, venha de onde vier.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]] · [[A regra mora fora da porta que a chama]]
- Irmã: [[Consumir recurso de uso único é UPDATE condicional, não checar antes]] · [[Permissão se valida no servidor, não na interface]]
- Visto em: [[NaveX]] (a Administração: usuário, cargo e exclusão passam pela mesma conferência)
- Mapa: [[Dados]]
