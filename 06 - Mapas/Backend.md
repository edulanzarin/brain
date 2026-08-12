---
tags: [tipo/moc]
criado: 2026-07-20
---

# Backend

Técnicas de servidor: Node, rotas, arquivos, integrações. Princípios em [[Base]];
o que é banco de dados tem mapa próprio em [[Dados]].

## Processos e arquivos

- [[Armadilhas de child_process no Node]] — timeout, stderr e `EPIPE`; por que
  `spawn` em vez de `exec`.
- [[Ler extrato bancário em PDF]] — extrair dado estruturado de PDF.
- [[Servir anexo por rota com checagem de permissão]] — arquivo protegido não é
  arquivo estático. Princípio: [[Permissão se valida no servidor, não na interface]].
- [[Trocar o backend de armazenamento sem downtime]] — mover binário do banco pra
  uma pasta com ponteiro, leitura de reserva e migração sob demanda. Princípio:
  [[Migração de dados mantém o antigo como reserva até a virada]].
- [[No pfx renovado o titular é a folha de validade mais recente]] — um `.pfx`
  embute a cadeia e às vezes o cert antigo; entre as folhas, escolher a de maior
  validade em vez da primeira. Princípio:
  [[Um invariante se garante na estrutura, não no processo]].

## Autenticação e permissão

- [[Cravar o seam de permissão antes do login]] — stub da sessão + gate num lugar só,
  pra permissão não virar retrofit. Princípio:
  [[Permissão se valida no servidor, não na interface]].
- [[Sessão opaca no banco separa autenticação de permissão]] — cookie só carrega o
  token; permissão vem do banco a cada request (revogável, sempre atual).
- [[Sessão de painel interno é um cookie assinado, não uma tabela de sessões]] — o
  outro extremo: painel de 1-2 pessoas, sessão stateless num cookie HMAC, sem store.
  Princípio: [[A assinatura autentica o dado, não quem o trouxe]].
- [[Escopo de dado se clampa no servidor, num funil só]] — a lista de "o que o
  usuário pediu" não é confiável; a restrição real mora num funil único.
- [[Drill-down por id foge do funil de escopo e precisa de gate próprio]] — a rota
  de detalhe por `?empresa=` não passa pelo funil; re-checar o dono do registro.
- [[O que dois módulos compartilham é a query, não a rota]] — reuso de dado entre
  módulos sem furar o gate por módulo.
- [[Formulário público por token opaco fica fora do gate de sessão]] — quem não
  tem login responde por link; o token é a credencial, a exceção ao gate é
  cirúrgica.
- [[Canal anônimo não guarda quem, e o retorno é um segredo do denunciante]] —
  quando o canal promete anonimato, não se grava identidade nenhuma; o retorno é
  um protocolo+senha (hash) que só o denunciante tem, e o agregado suprime recorte
  pequeno. Princípio: [[Um invariante se garante na estrutura, não no processo]].
- [[Uma resposta canônica de um grupo é um token compartilhado]] — vários podem
  responder, mas só uma resposta vale: um token para o grupo, o primeiro fecha.
  Princípio: [[Um invariante se garante na estrutura, não no processo]].
- [[Permissão composta por papéis somados, não exceção por usuário]] — o cargo
  concentra tudo; a pessoa acumula papéis e o acesso é a união, sem override por
  gente. Princípio: [[Permissão se valida no servidor, não na interface]].
- [[Posse numa permissão binária é duas seções e recorte por linha]] — "dono vê os
  seus, gestor vê todos" não é view/edit: duas seções + recorte por `autor` no
  servidor. Princípio: [[Permissão se valida no servidor, não na interface]].
- [[Filtro transversal só é honesto se todo o funil o honra]] — dimensão de
  filtro nova (filial) precisa chegar a toda consulta; funil compartilhado
  propaga, query própria é buraco silencioso.
- [[Criar e editar passam pelo mesmo funil de resolução]] — o lado da escrita: a
  regra que resolve dono/vínculo tem que rodar em criar E editar; edição com
  regra própria (ou nenhuma) aceita o que o cadastro recusa. Princípio:
  [[Um invariante se garante na estrutura, não no processo]].
- [[Dígito verificador rejeita o documento errado na entrada]] — CPF/CNPJ tem
  dígito verificador; conferir no funil barra o número digitado errado antes que
  ele crie a entidade errada. Princípio:
  [[Um invariante se garante na estrutura, não no processo]].
- [[Importação em massa passa pela API, não pelo banco]] — migrar mandando cada
  registro pelo endpoint de cadastro herda validação, dono e dedup; INSERT direto
  reimplementa tudo na mão e diverge. Princípio:
  [[Um invariante se garante na estrutura, não no processo]].
- [[Para alimentar o ERP, gere o arquivo de importação dele]] — o espelho, virado
  pra fora: pra escrever num sistema de terceiro (Questor), prepare o arquivo de
  importação dele em vez de INSERT no banco; ele aplica os invariantes na ingestão.
  Princípio: [[Um invariante se garante na estrutura, não no processo]].
- [[Atributo efetivo é o do dono, ou o local quando não há dono]] — atributo que
  mora numa entidade dona, mas o filho às vezes não tem dono: campo local opcional
  + efetivo `dono ?? proprio`, guardando o local só sem dono. Princípio:
  [[Um invariante se garante na estrutura, não no processo]].

## Composição e honestidade do resultado

- [[Fluxo de fechamento é orquestração dos motores que já existem]] — a tela de
  "posso fechar?" reusa os motores validados e só decide a cor; não recalcula.
- [[Fila de exceção recomputa o achado e persiste só a triagem]] — worklist sobre
  fonte read-only: recomputa os achados a cada carga e grava só a decisão humana
  (resolver/ignorar), com identidade estável. Princípio:
  [[Um invariante se garante na estrutura, não no processo]].
- [[Deixar o método da conferência visível quando o SQL não foi validado]] — heurística
  que não deu pra validar mostra a memória de cálculo, pra o humano com o dado validar.
- [[O que o Questor não dá mora no app-db chaveado pela identidade dele]] — dois pools
  (fonte read-only + app gravável); correção, entidade ausente e renomeação viram overlay
  no banco do app, chaveado pela identidade da fonte, com merge em TS. Princípio:
  [[Sobre fonte read-only, o editável mora no seu banco chaveado pela identidade dela]].

## IA e LLM

- [[Coleta determinística, LLM só interpreta]] — o código coleta e calcula tudo;
  o LLM recebe os dados prontos e só interpreta, nunca busca nem faz a conta.
- [[Censurar a identificação antes de mandar pro LLM externo]] — nome e CNPJ do
  cliente saem antes do envio (marcador na origem + scrub) e voltam só depois; a
  identificação não cruza a fronteira do terceiro.

## Integrações

- [[Polling substitui webhook quando não há IP público]] — quando não dá pra receber
  chamada de fora. Princípio: [[Configuração vem do ambiente, não do código]].
- [[Adapter de canal isola o app do provider de mensageria]] — provider externo trocável
  (WhatsApp: Baileys ou Cloud API) fica atrás de uma interface; o app não vê o fornecedor.
- [[Persistir a mensagem não espera a entrega, a entrega é status]] — gravar (durável,
  interno) e entregar (externo, falível) são passos separados; a entrega vira status da
  mensagem (`pendente → enviado → entregue → lida`), nunca pré-requisito da gravação.
- [[Webhook de provedor chega repetido e fora de ordem, a borda tolera os dois]] —
  entrega at-least-once e sem ordem: idempotência numa chave única, status que só avança,
  200 rápido. Princípio: [[Um invariante se garante na estrutura, não no processo]].
- [[Webhook de terceiro se valida pela assinatura antes de confiar no corpo]] — URL
  pública é hostil; conferir o HMAC sobre o corpo cru com o segredo do app antes do parse.
- [[Casar telefone brasileiro tolerando o nono dígito]] — o mesmo celular tem duas grafias
  (com/sem o 9); casar por classe de equivalência pra não duplicar contato na ingestão.
- [[De-para determinístico com override que vira aprendizado]] — casar conta/código
  de dois sistemas por cascata (chave → estrutura → descrição restrita), sem IA; a
  correção do humano vira override salvo e o de-para melhora a cada uso. Princípio:
  [[A definição em dado dirige o comportamento, não um caso no código]].

## Cabeçalhos HTTP

- [[CSP só aparece no build de produção, toda origem externa vai no allowlist]] —
  o dev server não tem CSP; recurso externo (fonte, tile) só quebra em prod.
  Princípio: [[Verificar no build de produção, não só em dev]].

## Princípios que mandam aqui

- [[Permissão se valida no servidor, não na interface]]
- [[Configuração vem do ambiente, não do código]]
- [[Ambiente de dev sobe igual ao de produção]]
- [[Um invariante se garante na estrutura, não no processo]]

---

Voltar para [[Início]]
