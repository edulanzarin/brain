---
tags: [tipo/atomica, camada/padrao, dev/backend, sql]
criado: 2026-07-25
---

# Filtro transversal só é honesto se todo o funil o honra

> Ao adicionar uma DIMENSÃO de filtro a um sistema (empresa, filial, período…),
> ela precisa ser aplicada por TODA consulta que a dimensão recorta — senão a
> tela mostra um controle que filtra umas coisas e ignora outras, e o usuário
> não sabe quais. Um filtro que mente é pior que filtro nenhum.

O barato: onde as consultas passam por um **construtor de WHERE compartilhado**
(um `buildWhere` só, o "funil"), a dimensão nova entra num lugar e **propaga de
graça** para todas elas. Foi assim que a filial (`codigoestab`) chegou a todo o
Fiscal de uma vez — uma condição no funil, e as ~30 rotas honraram.

O caro: as consultas que montam o **WHERE próprio** (as pesadas/especiais, tipo
uma reconciliação) são **buracos silenciosos** — não veem a dimensão nova a
menos que você costure `and codigoestab = any(...)` em CADA scan. Pior quando a
consulta compara dois lados (esperado × real): filtrar só um lado desalinha o
resultado. Regra: filtre TODOS os scans de fato, ou nenhum.

Duas decisões que caem daqui:
- **Prefira rotear pelo funil.** Query fora do funil é dívida que reaparece a
  cada dimensão nova.
- **Só mostre o controle onde ele é honrado.** Se metade do módulo ainda não
  costurou a dimensão, desligue o seletor nele até costurar — entregue o módulo
  inteiro de uma vez, não um controle pela metade. Verifique com o teste da
  soma: **filtro "todas" tem que bater com o consolidado de antes** (regressão),
  e cada recorte é subconjunto.

## Cópia do funil é buraco com cara de funil

O buraco mais difícil de ver é a consulta que tem o funil copiado em vez de
chamado. Ela recorta pelo escopo da sessão, passa em toda revisão de segurança e
parece honrar o filtro, até o dia em que o funil central ganha uma entrada nova.
A cópia fica com a regra antiga, sem erro.

No DP, a produtividade e as rescisões tinham cada uma a sua função de escopo,
escrita antes de existir grupo de empresas. Quando o grupo entrou no funil do
Fiscal, as duas continuaram lendo só a lista de empresas: escolher um grupo no
topo mostrava o escritório inteiro. A correção foi apagar as cópias e chamar o
funil.

Para achar as cópias, procure quem lê a permissão da sessão fora do funil
(`empresasPermitidas` no caso). Cada ocorrência é candidata: se ela também recebe
a lista de empresas do cliente, é uma cópia; se só monta painel ou lista de
opções sem pedido do cliente, é leitura legítima. No NaveX sobraram só as
legítimas depois da correção.

Visto em: [[NaveX]] (as cópias do DP) · [[Navetech Hub]] — filial por `codigoestab`: Fiscal (via buildWhere)
saiu na hora; Contábil (Conferência/Balancete, query própria) ficou desligado
até costurar. Relaciona com [[Escopo de dado se clampa no servidor, num funil só]]
(o mesmo funil que clampa empresa é onde a filial entra).
