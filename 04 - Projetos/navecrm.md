---
tags: [tipo/projeto, projeto/navecrm]
criado: 2026-09-01
status: ativo
codigo_em: ~/Dev/navecrm
---

# navecrm

> CRM de atendimento por WhatsApp para escritórios de contabilidade, multi-inquilino
> numa instância só. Isolamento entre escritórios feito pelo **banco** (RLS, não filtro
> na consulta) e conector de WhatsApp de verdade por Baileys, com a sessão no Postgres
> e um freio de envio.

Código em: `~/Dev/navecrm` · sem remote ainda (commit local em dia).

## De onde veio

Escrito do zero em set/2026, a pedido explícito do Eduardo, depois de duas tentativas
no mesmo problema: o [[CRM Contábil]] (arquivado no mesmo dia) e uma base anterior
monotenant. **Nada foi portado de outro projeto** — a instrução era essa, e vale
registrar porque a tentação de reaproveitar foi o que atrapalhou a rodada anterior.

## Estado atual

De pé e verificado no compose completo (app, banco, migração e robô):

- Nove telas lendo do Postgres: Hoje, Conversas, Clientes, Contatos, Tarefas,
  Arquivos, Números, Ajustes e o catálogo. Build de produção e tipos limpos.
- **Isolamento provado por script**, não por leitura: cinco travessias, incluindo a
  que reprova `SET` sem `LOCAL` (contexto que sobrevive ao commit).
- **WhatsApp fechando o laço dentro do Docker**: o app pede o pareamento ao robô pela
  rede do compose, o Baileys conecta e o QR real aparece no banco. Falta só alguém ler
  com o celular.
- Login real (scrypt, sessão opaca no banco) e semente com **dois** escritórios de
  propósito — um grande com tudo ligado, outro pequeno com metade dos módulos — para a
  modularidade ser verificável em vez de declarada. Confirmado ao vivo: a mesma rota
  `/arquivos` responde 200 num escritório e 404 no outro.
- Escopo por setor conferido ao vivo: o atendente do Fiscal não alcança a conversa do
  Departamento Pessoal nem digitando o endereço dela.

## Infra

Slug `navecrm` · app `navecrm-app` na `4078` · banco `navecrm-db` na `5078` · robô
`navecrm-robo` **sem porta publicada** (o app fala com ele pela rede interna). Chassi e
mapa em [[Infra]].

## Stack

Next 15.5 (App Router, Server Actions) · React 19 · TypeScript · Postgres 17 em SQL
puro com `pg` · Tailwind v4 · Baileys 6.7 num processo à parte, rodado por `tsx`.

## Decisões importantes

- **O inquilino se isola no banco, não na aplicação** —
  [[Isolamento entre clientes é política do banco, não filtro na query]].
- **Modularidade em dado, não em código.** Módulo ligado é linha, setor é linha, e
  campo que só um escritório usa é definição com valor no `jsonb` da entidade.
  Escritório novo não pede deploy. Mesma família de
  [[Formulário montado pelo usuário — a definição no banco dirige renderer e validação]].
- **Módulo desligado responde 404**, não some só do menu.
- **Dois funis, um dentro do outro.** A política responde de qual escritório é a linha;
  dentro dele, um funil na aplicação responde quais setores a pessoa alcança —
  [[Escopo de dado se clampa no servidor, num funil só]].
- **O cabeçalho de cada tela é componente de servidor, não portal.** Foi decisão de
  partida, e não descoberta no caminho:
  [[Barra de topo contextual - o módulo injeta suas ferramentas via portal]] traz o
  porquê.
- **O robô é processo separado** porque a conexão é estado vivo —
  [[Processo que segura sessão viva não morre em exceção não tratada]].
- **A fila é a costura** entre o app e o WhatsApp —
  [[Persistir a mensagem não espera a entrega, a entrega é status]].
- **As regras do freio moram num módulo compartilhado**, porque são lidas em dois
  processos: o robô aplica, a tela mostra o teto de hoje com a mesma conta.
- **A cor do escritório vira quatro valores computados** —
  [[Cor de marca precisa de variante acessível por tema]].

## Aprendizados (viraram notas)

- [[Isolamento entre clientes é política do banco, não filtro na query]]
- [[Contexto de tenant tem que morrer no commit, senão o pool o carrega adiante]]
- [[Uma conexão do pg não atende duas consultas ao mesmo tempo]]
- [[Binário guardado em JSON volta como objeto de números]]
- [[Em canal humano automatizado, o ritmo denuncia antes do volume]]
- [[Barra de topo contextual - o módulo injeta suas ferramentas via portal]]

## Próximos passos

- [ ] Ler o QR com um número de teste e trocar mensagem de verdade ponta a ponta.
- [ ] Cadastro de cliente, contato e tarefa (as telas leem; não há botão de criar).
- [ ] Recebimento de mídia (o robô ignora anexo; a tabela `anexo` já espera).
- [ ] Tempo real na fila (hoje a tela recarrega).
- [ ] Remote e primeiro push.

## Conexões
- Usa: [[Design]] · [[Infra]] · [[Backend]] · [[Dados]]
- Substitui: [[CRM Contábil]]
- Base: [[Permissão se valida no servidor, não na interface]] · [[Configuração vem do ambiente, não do código]] · [[Ambiente de dev sobe igual ao de produção]]
- Mapa: [[Projetos]]
