---
tags: [tipo/moc]
criado: 2026-07-20
---

# Dados

Modelagem, query e performance em banco. Aqui ficam as técnicas **gerais**; o schema
de sistema externo é referência e tem mapa próprio.

## Postgres

`02 - Técnicas/Banco de dados`

- [[Isolamento entre clientes é política do banco, não filtro na query]] — multi-tenant
  por RLS: a consulta que esquecer o filtro devolve zero linhas em vez das do vizinho,
  porque vazamento de tenant não tem sintoma. Princípio:
  [[Permissão se valida no servidor, não na interface]].
- [[Contexto de tenant tem que morrer no commit, senão o pool o carrega adiante]] — o
  `SET` sem `LOCAL` deixa a conexão carimbada e o próximo pedido lê o cliente anterior;
  intermitente por construção, e a política de isolamento não pega.
- [[Uma conexão do pg não atende duas consultas ao mesmo tempo]] — `Promise.all` dentro
  da transação promete concorrência e entrega fila; some numa atualização de dependência.
- [[Acesso comprado é linha própria, não status do pedido]] — "pagou" e "tem acesso"
  parecem a mesma pergunta e não são; matrícula em tabela separada, com ponteiro
  opcional pro pedido, aguenta reembolso, cortesia e curso gratuito. Princípio:
  [[Um invariante se garante na estrutura, não no processo]].
- [[Agregar antes de juntar em tabelas gigantes no Postgres]] — reduzir antes de
  juntar; o padrão que salvou consulta em tabela de 47M linhas.
- [[Estoque e fluxo numa série a partir de datas de início e fim]] — de datas de
  início/fim saem fluxo (entrou/saiu) e estoque (ativos numa data); série densa
  com `generate_series` + `count filter`.
- [[Formulário montado pelo usuário — a definição no banco dirige renderer e validação]]
  — campos tipados + `config` jsonb por tipo; uma peça só renderiza, pré-visualiza e
  valida. Princípio: [[A definição em dado dirige o comportamento, não um caso no código]].
- [[Entidade núcleo cresce por tabela satélite, não por coluna]] — mantém a tabela central
  mínima; feature nova é tabela que referencia, não coluna adicionada. Escala por composição.
- [[Registro que muda de casa leva junto o token já distribuído]] — na migration que
  promove um modo escondido a entidade própria, o identificador que o mundo lá fora
  segura (o link já enviado) vem junto; ele também é a ponte entre as duas tabelas.
  Princípio: [[Migração de dados mantém o antigo como reserva até a virada]].
- [[Campo que vira indicador é coluna, o resto do documento é jsonb]] — num documento de
  forma fixa, o que se agrega/filtra vira coluna; narrativa e listas variáveis vão em jsonb.
- [[Grão fino numa varredura só dispensa os count distinct]] — painel que quebra o
  mesmo fato por vários eixos: agrupe uma vez pelo grão mais fino da tela (2M de
  linhas viram 6 mil) e faça ranking, série, distintos e calendário em memória.
  Princípio: [[Reduzir a cardinalidade vem antes de enriquecer]].
- [[Percentil ponderado sai do grão agregado, sem segunda varredura]] — se o grão já é
  (valor, quantas vezes), mediana e p90 saem dele em memória, exatos e por todos os
  eixos de uma vez; `percentile_disc` precisaria das linhas e de uma varredura por eixo.
  Princípio: [[A régua sai da distribuição, não dos extremos]].
- [[Numeric e bigint do Postgres chegam como string no driver pg]] — o `node-pg`
  entrega `numeric`/`bigint` como string; castar pra `float8` pra receber number.
- [[O agrupamento útil sai do campo que o operador preenche]] — quando o schema tem o
  campo normalizado e o campo livre para a mesma dimensão, quem agrupa é o livre: é lá
  que quem opera escreve a taxonomia real. Reserva no `coalesce`, e cardinalidade que
  cresce com as linhas é o sinal de que se escolheu o campo errado.
- [[Consumir recurso de uso único é UPDATE condicional, não checar antes]] —
  cupom/vaga/estoque de um: o `WHERE estado_livre` no UPDATE decide a corrida pelo
  `rowCount`, sem lock. Princípio: [[Um invariante se garante na estrutura, não no processo]].

## Referência de schema externo

- [[Banco Questor]] — o banco do ERP contábil, com todos os módulos. É **referência**,
  não técnica: descreve um sistema de terceiro que eu não controlo.

## Migrations

Migration é infra, não banco: [[Migrations em container próprio no Docker Compose]],
mapa [[Infra]].

- [[Renomear coluna é migration à mão; a gerada derruba e recria]] — o diff do ORM não
  lê intenção, e o dado da coluna renomeada é justamente o que o mundo lá fora segura.
- [[Registro que muda de casa leva junto o token já distribuído]] — mesma família: o
  identificador atravessa a migration junto com a linha.

## Princípios que mandam aqui

- [[A unidade de contagem é o ato, não a linha que ele deixou]] — antes de contar,
  pergunte quantas linhas um ato gera; se não for uma, agrupe pela chave do lote e
  deixe a contagem de linhas como número secundário.
- [[Identificador que já circulou não é mais seu para mudar]] — chave que virou link,
  protocolo ou endereço na mão de terceiro não é mais detalhe de implementação.
- [[Reduzir a cardinalidade vem antes de enriquecer]] — a ordem das duas metades de
  toda consulta sobre tabela enorme: reduzir e só depois enriquecer.
- [[Plataforma de IA hospedada prende o app pelo banco]] — o banco é o que realmente
  prende um app a um fornecedor.

---

Voltar para [[Início]]
