---
tags: [tipo/atomica, camada/padrao, seguranca, sql, dev/backend]
criado: 2026-09-01
---

# Isolamento entre clientes é política do banco, não filtro na query

> Quando uma instância atende vários clientes (escritórios, empresas, contas), o
> `WHERE cliente_id = ...` em toda consulta é **disciplina**, e disciplina falha na
> consulta número quarenta. Row Level Security move a regra para o armazenamento: a
> consulta que esquecer o filtro devolve zero linhas em vez de devolver as do vizinho.

## O erro que evita

O vazamento multi-tenant não tem sintoma. A query sem o filtro **funciona** — retorna
linhas, a tela renderiza, ninguém percebe. Só que são as linhas de outro cliente. Não
há erro no log, não há exceção, não há teste que quebre, porque do ponto de vista do
banco a consulta estava correta.

Com política, o mesmo esquecimento vira lista vazia: um bug visível, que alguém abre
chamado sobre, no mesmo dia.

## A costura

Três peças, e a terceira é a que costuma faltar:

```sql
-- 1. o contexto, com falha fechada: sem valor, NULL, e NULL não casa com nada
CREATE FUNCTION app_escritorio() RETURNS uuid LANGUAGE sql STABLE AS $$
  SELECT NULLIF(current_setting('app.escritorio_id', true), '')::uuid
$$;

-- 2. a política, em laço sobre toda tabela que tem a coluna
ALTER TABLE contatos ENABLE ROW LEVEL SECURITY;
ALTER TABLE contatos FORCE  ROW LEVEL SECURITY;
CREATE POLICY tenant ON contatos
  USING (escritorio_id = app_escritorio())
  WITH CHECK (escritorio_id = app_escritorio());
```

3. **A aplicação entra por um papel que não é dono das tabelas e não tem `BYPASSRLS`.**
Sem isso o resto é enfeite: dono de tabela ignora política (daí o `FORCE`), e
superusuário ignora as duas coisas. Migration e semente rodam pelo dono justamente
porque precisam atravessar.

O `WITH CHECK` é metade do valor: sem ele a leitura fica isolada mas a **escrita** não,
e o sistema aceita gravar uma linha no cliente vizinho.

## O que mais vale lembrar

- **O tenant é a primeira coluna de todo índice.** Com RLS, índice que não começa por
  ele faz a consulta varrer a tabela inteira antes de filtrar.
- **As consultas que acontecem ANTES de haver contexto** — login, troca de cookie por
  identidade, o worker descobrindo o que levantar — não podem passar pela política.
  Vão em funções `SECURITY DEFINER` de recorte fechado, que não aceitam filtro livre.
  São poucas, e listá-las é uma boa medida de saúde do desenho: se forem muitas, o
  contexto está sendo assumido tarde demais.
- **O runner de migration recusa terminar** se alguma tabela com a coluna do tenant
  ficou sem política. É o que impede a tabela quarenta e um de nascer aberta.
- Ele contrasta com [[Escopo de dado se clampa no servidor, num funil só]] e não o
  substitui: o funil recorta **dentro** do cliente (quais empresas este usuário vê) e
  continua sendo trabalho da aplicação. A política responde por uma pergunta anterior,
  que é de qual cliente é a linha — e a resposta a essa não pode depender de ninguém
  lembrar.

## Conexões
- Princípio: [[Permissão se valida no servidor, não na interface]]
- Irmã: [[Escopo de dado se clampa no servidor, num funil só]] · [[Contexto de tenant tem que morrer no commit, senão o pool o carrega adiante]]
- Visto em: [[CRM Contábil]]
- Mapa: [[Dados]] · [[Backend]]
