---
tags: [tipo/projeto, projeto/cofre-digital]
criado: 2026-07-20
status: ativo
codigo_em: ~/Dev/cofre-digital
---

# Cofre Digital

> Cofre da intranet Navecon: certificados digitais (e-CNPJ, e-CPF, NF-e),
> acessos a sites com manual ilustrado e alvarás — organizados por empresa,
> com permissão por setor e aviso de vencimento.

Código em: `~/Dev/cofre-digital`

## Estado atual

Rodando em Docker, acessível na rede em `http://<ip>:4004`. Três módulos de
conteúdo (certificados, acessos, alvarás) mais empresas, equipe e configurações.
Em jul/2026 passou por uma rodada de acabamento do design e, depois, pela
regularização da infra (ver abaixo).

## Infra (regularizada em 20/07/2026)

Primeiro projeto a receber o chassi do [[Infra]] por inteiro. Antes estava fora dele
em quase tudo: containers com sufixo do compose (`cofre-digital-db-1`), porta
`4004:3000` fixa no compose, `"dev": "next dev"` sem porta, volume `pgdata` genérico
e a porta do banco escondida num `docker-compose.override.yml` gitignorado — que
ainda por cima divergia do `.env.example` (5435 contra 5433).

Como ficou:

| Recurso | Nome | Porta no host |
|---|---|---|
| App | `cofre-digital-app` | `0.0.0.0:4004` → 3000 |
| Banco | `cofre-digital-db` | `127.0.0.1:5004` → 5432 |
| Migrations | `cofre-digital-migrate` | — |
| Rede | `cofre-digital-net` | — |
| Volume | `cofre-digital-db-data` | — |

O override sumiu: o bind `127.0.0.1` no compose principal dá o mesmo acesso local sem
expor o banco na rede e sem um segundo arquivo fora do repositório. Um compose só,
igual em dev e em produção — [[Ambiente de dev sobe igual ao de produção]].

A migração do volume (`cofre-digital_pgdata` → `cofre-digital-db-data`) foi feita com
`pg_dump` antes e cópia por container intermediário; os 5 certificados e 5 empresas
seguiram intactos. O volume antigo ficou parado como rede de segurança.

## O que tem

- **Certificados**: upload de `.pfx` com leitura automática do próprio arquivo
  (titular, CNPJ/CPF, tipo, AC, validade), A1 vs A3, download, anel de
  vencimento e histórico de eventos.
- **Acessos**: site, tipo de login, credenciais e **manual em Markdown** com
  imagens (upload ou Ctrl+V), com prévia ao escrever.
- **Alvarás**: vencimento opcional (existe alvará permanente), arquivo anexo e
  histórico. Os sem data entram no total mas ficam fora das barras de situação.
- **Empresas**: cada uma com seu próprio cofre, agregando os três módulos.
- **Bloqueio do cofre**: PIN, bloqueio manual e automático por inatividade —
  camada extra além do login.
- **Janela de "vencendo" configurável**, valendo para badges, filtros e dashboard.

## Decisões importantes

- **Permissão validada na API, não escondendo botão.** A senha nem chega ao
  navegador dos setores de leitura: só o endpoint de cópia a entrega.
- **Perfis de permissão por módulo** (`view`/`edit`), em vez de amarrar tudo ao
  setor — o setor virou rótulo, o perfil virou a regra.
- **Sem biblioteca de UI.** Design system próprio com prefixo `vlt-` em
  `globals.css`, seguindo o [[Design]]. Coerente com
  [[Manter o tooling enxuto e o conhecimento no cérebro]].
- **Dark-first**: o cofre nasce no modo noturno, claro é opcional — o contrário
  do padrão dos outros projetos.
- **Preferências pessoais no navegador** (tema, PIN, janela de alerta), dados no
  Postgres. Ninguém precisa sincronizar gosto pessoal entre máquinas.

## Rodada de design (jul/2026)

Três buracos entre o app e o sistema do [[Design]], todos fechados:

1. **24 `alert()` nativos** eram o único feedback do app — janela do sistema,
   fora da linguagem visual, e só para erro. Viraram toast, e as ações de
   sucesso passaram a confirmar. Ver [[Toast em vez de alert para o feedback do app]].
2. **Carregamento em bloco cinza** (19 retângulos pulsando, e no dashboard dois
   vãos em branco). Virou esqueleto com a forma do conteúdo. Ver
   [[Esqueleto de carregamento imita a forma do conteúdo]].
3. **Filtro fora da URL** — a página de certificados chegava a *limpar* a query
   string. Agora busca e filtro moram na URL. Ver [[Filtro de lista mora na URL]].

## Stack

Next.js 16 (App Router) · React 19 · TypeScript · Tailwind v4 · Prisma 7 ·
PostgreSQL 17 · Docker Compose. Ícones lucide, PKCS#12 lido no navegador.

## Aprendizados (viraram notas atômicas)

- [[Toast em vez de alert para o feedback do app]]
- [[Esqueleto de carregamento imita a forma do conteúdo]]
- [[Filtro de lista mora na URL]]
- [[Portal condicional dispensa o flag de montagem]]
- [[NoInfer faz o genérico sair da lista, não do valor padrão]]
- [[Semear teste cria linha nova, não muta linha real]] — aprendido do jeito
  ruim aqui: a verificação da coluna de grupo apagou uma vinculação real.
- [[Entidade auxiliar se cria no ponto de uso, não em tela própria]] — a aba Grupos
  virou criação inline no formulário e gestão no filtro.
- [[Seletor cria e gerencia os próprios itens]] — o Combobox que criou/renomeou/
  excluiu grupo sem tela dedicada.
- [[Migração de dados mantém o antigo como reserva até a virada]] — arquivos
  saindo do banco pra uma pasta sem corte seco.
- [[Trocar o backend de armazenamento sem downtime]] — a mecânica: ponteiro,
  leitura de reserva, migração sob demanda.
- [[Trocar o arquivo repede a senha e relê os dados]] — o `.pfx` renovado com
  senha diferente que salvava errado; leitura vira parte de escolher o arquivo.
- [[No pfx renovado o titular é a folha de validade mais recente]] — o `.pfx`
  renovado que trazia o cert antigo junto e continuava marcando "Vencido".
- [[Criar e editar passam pelo mesmo funil de resolução]] — a edição não resolvia
  a empresa dona pelo CNPJ como o cadastro; trocar por um `.pfx` de outro CNPJ
  deixava o cert na empresa errada. Virou uma regra só (`resolveCertCompany`).

## Grupos de empresas (jul/2026)

Grupo econômico: um punhado de empresas do mesmo dono, usado como eixo de filtro
(lista de empresas e de certificados, com a coluna "Grupo" — o certificado carrega
o nome do grupo da empresa dona pelo `CERT_INCLUDE`). O filtro mora na URL
(`?grupo=`) como os outros, num Combobox com busca.

Começou com uma **aba Grupos** própria (`/grupos`) pra cadastrar à mão. A aba foi
**removida**: o grupo passou a nascer e ser gerenciado de onde já é usado, virando
o primeiro caso de [[Entidade auxiliar se cria no ponto de uso, não em tela própria]].
Como ficou:

- **Nasce no formulário** da empresa e do certificado: digita um nome que não existe
  no seletor e ele é criado na hora. No cert isso resolve o atrito real — o cert
  sobe, puxa a empresa automática, e o grupo entra na mesma tela (atribui à empresa
  dona, nunca desvincula; vazio não mexe).
- **Renomeia/exclui dentro do próprio filtro** de grupo (lápis/lixeira no painel).
- **Grupo sem empresa é podado no servidor** após qualquer mutação que possa
  esvaziá-lo — some sozinho, sem faxina manual.

O Combobox que sustenta isso virou técnica: [[Seletor cria e gerencia os próprios itens]].

## Arquivos em pasta na rede (jul/2026)

Os binários (o .pfx do certificado, o PDF do alvará, a imagem de print) eram
base64 dentro do Postgres. Passaram a poder morar numa **pasta raiz** (um
compartilhamento de rede montado no container) — dado estruturado fica no banco,
arquivo vai pro disco. A pasta se define em Configurações, atrás de duas travas:
admin e uma senha que vive só na env `STORAGE_ROOT_PASSWORD` do servidor.

Feito aditivo e sem susto (o cofre acabava de perder grupos por auto-limpeza):
`filePath` novo ao lado do `fileData` antigo, gravação nova no disco, leitura
disco-primeiro-banco-reserva, e um migrador sob demanda que só zera o base64
depois de gravar o arquivo. Sem pasta configurada, tudo segue no banco. Virou a
técnica [[Trocar o backend de armazenamento sem downtime]] e o princípio
[[Migração de dados mantém o antigo como reserva até a virada]] — cujo segundo
caso é a própria migração de volume Docker deste projeto (dump antes, volume
antigo parado como rede).

## Passada de copy (jul/2026)

Depois do armazenamento, uma varredura de texto no app inteiro para um tom de
sistema de cliente: fora todo travessão de frase, fora o jargão de backend que
tinha vazado pra tela (o campo de pasta falava de `.env`, container, banco) e
corte dos textos que só explicavam funcionalidade (subtítulos de modal de
cadastro, toasts que narravam efeito). Virou o pensamento
[[Interface de cliente fala pouco e esconde o backend]].

## Edição de certificado (jul/2026)

Editar certificado dava "erro ao salvar" em produção. Três problemas se somavam, e
a correção mexeu nos três:

- **Senha herdada do estado antigo.** Trocar por um `.pfx` renovado (senha
  diferente) mantinha a senha antiga no formulário e salvava com ela. Agora
  escolher o arquivo abre um modalzinho que pede a senha e lê os dados na hora — o
  arquivo só é adotado depois de a senha decifrá-lo, e o botão "Ler dados" solto
  sumiu. Virou [[Trocar o arquivo repede a senha e relê os dados]].
- **Regravava o arquivo à toa.** O form recarregava os bytes do `.pfx` e os
  reenviava a cada save, fazendo toda edição reescrever o arquivo na rede (SMB via
  smbclient) — mesmo mudando só uma observação. Agora os bytes só voltam quando o
  arquivo muda de fato.
- **Erro real escondido.** O `catch` do PUT devolvia "Certificado não encontrado"
  pra qualquer falha, inclusive a de gravação na rede. Agora registra no servidor
  e devolve a causa — segundo caso de [[Chamada externa tem timeout e erro tratado]]
  (capturar o canal de erro em vez do genérico).

## Próximos passos possíveis

- Skeleton no refetch em vez de só no primeiro load, se o volume crescer.
- Exportar a lista filtrada (o filtro já está na URL, então o link já serve).

## Conexões

- Usa: [[Design]]
- Usa: [[Infra]]
- Faz parte de: [[Projetos]]
- Mapa: [[Projetos]]
