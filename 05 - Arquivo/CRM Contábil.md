---
tags: [tipo/projeto, projeto/contabil-crm]
criado: 2026-09-01
status: arquivado
codigo_em: ~/Dev/contabil-crm
---

# CRM Contábil

> CRM de atendimento por WhatsApp para escritórios de contabilidade, multi-tenant numa
> instância só. O que ele tem de próprio: **isolamento entre escritórios feito pelo
> banco** (RLS, não filtro na query) e um **conector de WhatsApp de verdade** por
> Baileys, com a sessão no Postgres e um portão anti-ban.

Código em: `~/Dev/contabil-crm` · sem remote ainda (commit local em dia).

## Encerrado em set/2026

Substituído pelo [[navecrm]], escrito do zero no mesmo dia a pedido do Eduardo. A
decisão de "seguir nesta base" durou horas: o que ele queria era um sistema novo, e o
que eu fiz em vez disso foi portar o modelo de permissão do Navehub para cá — ou seja,
reaproveitar, que era exatamente o que ele não tinha pedido.

O código continua em `~/Dev/contabil-crm` até ele mandar apagar. O conhecimento não
vem junto: as notas atômicas que saíram daqui continuam valendo por si, e apontam para
o [[navecrm]] como evidência viva.

## A sobreposição que deu origem a isto

Este sistema nasceu em cima de um espaço que já estava ocupado — havia uma base de
agosto/2026 mais adiantada em produto e sem conector de WhatsApp nenhum, e uma
anterior, monotenant. Em set/2026 o Eduardo decidiu: **esta é a base**, e o que valer
a pena das outras vem para cá. O primeiro transplante já aconteceu (a supervisão
escopada ao setor).

Não há link de substituição porque nada foi substituído ainda: a regra "projeto não
linka projeto" só abre quando a relação é real.

## Estado atual

De pé e verificado, no compose completo (app, banco, migrations e worker):

- Seis telas lendo do Postgres: Início, Mensagens, Empresas, Contatos, Tarefas,
  Documentos, Relatórios e Configurações. Build de produção e typecheck limpos.
- **Isolamento provado por script**, não por leitura: cinco travessias, incluindo a que
  reprova `SET` sem `LOCAL` (contexto que sobrevive ao commit).
- **WhatsApp fechando o laço dentro do Docker**: o app pede pareamento ao worker pela
  rede do compose, o Baileys conecta e o QR real aparece no banco. Falta só alguém ler
  o código com o celular.
- Login real (scrypt, sessão opaca no banco), semente com **dois** escritórios de
  propósito — um grande com tudo ligado, outro pequeno com metade dos módulos — para a
  modularidade ser verificável em vez de declarada.
- Catálogo de componentes em `/sistema`.

## Infra

Slug `contabil-crm` · app `contabil-crm-app` na `4077` · banco `contabil-crm-db` na
`5077` · worker `contabil-crm-wa` **sem porta publicada** (o app fala com ele pela rede
interna). O par 4076/5077 não serviu porque o 4076/5076 já estava tomado pelo privello2,
que não constava na tabela. Chassi e mapa em [[Infra]].

## Stack

Next 15.5 (App Router, Server Actions) · React 19 · TypeScript · Postgres 17 em SQL
puro com `pg` · Tailwind v4 · Baileys 6.7 num processo à parte, rodado por `tsx`.

## Decisões importantes

- **O tenant se isola no banco, não na aplicação.** Toda tabela de domínio carrega
  `escritorio_id` e tem política; a aplicação entra por um papel sem posse e sem
  bypass. O runner recusa terminar se alguma tabela ficou sem política —
  [[Isolamento entre clientes é política do banco, não filtro na query]].
- **Modularidade em dado, não em código.** Módulo ligado é linha, departamento é linha,
  e campo que só um escritório usa é definição em `campos_personalizados` com valor no
  `jsonb` da entidade. Escritório novo não pede deploy. Mesma família de
  [[Formulário montado pelo usuário — a definição no banco dirige renderer e validação]].
- **Módulo desligado responde 404**, não some só do menu.
- **Dois funis, um dentro do outro.** A política de RLS responde de qual escritório é a
  linha; dentro do escritório, um funil na aplicação responde quais departamentos a
  pessoa alcança. Perguntas diferentes, camadas diferentes —
  [[Supervisão é papel do setor, não cargo global]].
- **O worker é processo separado** porque a conexão do WhatsApp é estado vivo e morreria
  a cada deploy dentro do Next — [[Processo que segura sessão viva não morre em exceção não tratada]].
- **A fila é a costura** entre o app e o WhatsApp: o app grava pendente e enfileira,
  quem decide o instante é o portão — [[Persistir a mensagem não espera a entrega, a entrega é status]].
- **A cor do escritório vira quatro valores computados** (preenchimento e texto, claro e
  escuro), validados por contraste — [[Cor de marca precisa de variante acessível por tema]].
- **A paleta do gráfico é token de dado, separada do acento da interface**, e passou nas
  seis checagens nos dois temas — [[Acento da interface é um token separado da cor de dado]].

## Aprendizados (viraram notas)

Só links; o texto mora na nota atômica.

- [[Isolamento entre clientes é política do banco, não filtro na query]]
- [[Contexto de tenant tem que morrer no commit, senão o pool o carrega adiante]]
- [[Uma conexão do pg não atende duas consultas ao mesmo tempo]]
- [[Binário guardado em JSON volta como objeto de números]]
- [[Em canal humano automatizado, o ritmo denuncia antes do volume]]
- [[Uma faixa de portas por projeto]] — ganhou o caso do projeto que ocupa porta sem
  estar na tabela, e o que isso custa.
- [[Verificar no build de produção, não só em dev]] — dois defeitos que passaram em dev:
  handler de clique em componente de servidor, e `Promise.all` sobre a mesma conexão.
- [[Supervisão é papel do setor, não cargo global]]
- [[Barra de topo contextual - o módulo injeta suas ferramentas via portal]] — ganhou o
  preço do padrão: o topo injetado por portal não existe no HTML do servidor, então
  título, abas e busca só aparecem depois do JavaScript. O remédio foi tirar a barra do
  layout e devolvê-la ao módulo.

## Próximos passos

- [x] **Destino decidido** (set/2026): esta é a base; o resto vem para cá.
- [x] Permissão escopada ao departamento, com supervisão por setor e prova das
      travessias — [[Supervisão é papel do setor, não cargo global]].
- [ ] Ler o QR com um número de teste e trocar mensagem de verdade ponta a ponta.
- [ ] CRUD de empresa, contato e tarefa (as telas leem; os botões de criar não existem).
- [ ] Recebimento de mídia (o worker ignora anexo; a tabela de documentos já espera).
- [ ] Tempo real na fila (hoje a tela recarrega).
- [ ] Remote e primeiro push.

## Conexões
- Usa: [[Design]] · [[Infra]] · [[Backend]] · [[Dados]]
- Base: [[Permissão se valida no servidor, não na interface]] · [[Configuração vem do ambiente, não do código]] · [[Ambiente de dev sobe igual ao de produção]]
- Mapa: [[Projetos]]
