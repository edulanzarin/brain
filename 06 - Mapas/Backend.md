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
- [[Auth.js sem adapter, a sessão JWT resolve o usuário no seu próprio SQL]] — login
  pronto (Google + email/senha) sem adapter de ORM: sessão JWT e os callbacks fazendo
  o upsert do usuário no `pg` cru. Gate no server component, não no middleware edge.
- [[Cookie de sessão é host-only, www e apex canonizam num domínio só]] — login no
  `www` não vale no apex (`__Host-` proíbe `domain=`); um host canônico, o outro só
  redireciona por host, `APP_URL` aponta pro canônico.
- [[Descobrir o shard por sondagem paralela com early-exit e cachear]] — quando o
  servico sharded nao diz qual no e seu, abre todos em paralelo, fica com o que
  responde primeiro e cacheia; o normal vira 1 conexao.
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

- [[Tier é nota com régua fixa, não posição na fila]] — faixa cortada por percentil de
  posição mede fila, não mérito: o corte sai de score calibrado no histograma e escrito
  como constante, senão um patch que melhora trinta itens não promove nenhum. Princípio:
  [[Ordene pela grandeza que decide, não pela que impressiona]].
- [[Campo que a normalização não copia vira número errado, não erro]] — o normalizador é
  o contrato com a fonte: campo raro que ninguém listou some sem log e o motor passa a
  mentir. Varra as chaves do conjunto todo a cada ingestão. Princípio:
  [[Auditar o registro, não só o agregado]].

- [[Fluxo de fechamento é orquestração dos motores que já existem]] — a tela de
  "posso fechar?" reusa os motores validados e só decide a cor; não recalcula.
- [[Regra de envio recorrente materializa uma campanha e reprograma]] — regra =
  formulário + público + frequência + ponteiro; o cron resolve o público de hoje,
  reusa o disparo manual e avança a data. Princípio:
  [[Recorrência guarda a receita e o próximo disparo, não N ocorrências futuras]].
- [[Fila de exceção recomputa o achado e persiste só a triagem]] — worklist sobre
  fonte read-only: recomputa os achados a cada carga e grava só a decisão humana
  (resolver/ignorar), com identidade estável. Princípio:
  [[Um invariante se garante na estrutura, não no processo]].
- [[Deixar o método da conferência visível quando o SQL não foi validado]] — heurística
  que não deu pra validar mostra a memória de cálculo, pra o humano com o dado validar.
- [[Distribuição exata sai de programação dinâmica, não de Monte Carlo]] — "quantos
  sorteios até somar N?" fecha por cadeia absorvente quando os ganhos são poucos e
  múltiplos de um mesmo passo; troque a unidade pelo mdc, some em inteiro e a mediana,
  o p90 e o desperdício saem da mesma varredura. Monte Carlo vira o teste, não o motor.
  Princípio: [[Custo de processo aleatório se orça pela cauda, não pela média]].
- [[O cálculo puro sai do módulo server-only para poder ser testado]] — a regra de
  negócio vive num módulo puro (sem DB, sem `server-only`) que o servidor orquestra;
  fica testável num runner e reusável em mais de um caminho. Princípio:
  [[Um invariante se garante na estrutura, não no processo]].
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

- [[Recusa não é falha: contra o não do servidor, insistir é ruído]] — retry serve pra
  falha passageira; contra recusa (banido, sem permissão, cota) insistir só gasta
  tentativa e esconde o motivo. Classifique 403/401/429 antes de decidir o retry, guarde
  a frase crua do outro lado e desligue a INTENÇÃO, não só a conexão. Princípio:
  [[Chamada externa tem timeout e erro tratado]].
- [[O bundle público do cliente entrega o contrato da API sem documentação]] — sem doc
  e sem HAR da ação, os chunks JS públicos do cliente oficial têm a URL e o payload
  literais; extrair de lá é contrato real, não palpite.
  Princípio: [[Um invariante se garante na estrutura, não no processo]].
- [[Quando a REST não expõe o dado, o WebSocket do mesmo sistema entrega]] — a
  superfície real é maior que a REST documentada; o dado que falta num canal está
  noutro (WS/SSE/gRPC), ao custo da sessão viva. Irmã do shard por sondagem.
- [[Sonda que falhou não é sinal de que mudou]] — a pergunta barata que decide se vale
  buscar o volume tem TRÊS respostas, e quase toda implementação prevê duas: `catch`
  devolvendo `null` faz qualquer falha virar download completo, em silêncio. Traz junto
  o log por transição, a cadência como variável própria e o piso de plausibilidade na
  resposta. Princípio: [[Peça o que a fonte mostra, não o que você precisa]].
- [[Config que a sessão cacheia no init não vê a escrita no backend, reaplique na mesma conexão]] —
  sessão longa lê a config no init e não vê a escrita posterior; reaplique reenviando o
  init na mesma conexão (não reconecte), e dispare isso do caminho de escrita.
  Princípio: [[Um invariante se garante na estrutura, não no processo]].
- [[Confirme a mutação pelo estado que ela deixa, não pelo ack que pode não chegar]] —
  comando que muta o outro lado (WS/fila/RPC): confirme relendo o estado e vendo o efeito,
  não esperando o ack que pode não vir. Princípio: [[Um invariante se garante na estrutura, não no processo]].
- [[Comando que responde ok e não muda nada tem pré-condição de estado]] — o ack veio e o
  estado ficou igual: o comando é de outro estado do sistema. Sai do estado, age, volta
  pela intenção guardada — e mostra na tela que está se recuperando.
  Princípio: [[Guarde a intenção e o processo se reconstrói dela]].
- [[Total ao vivo é o persistido fechado mais o em andamento ainda não gravado]] — dashboard
  cumulativo que não zera durante a unidade: soma o persistido (fechado) + o vivo (em memória),
  contando o vivo só enquanto não foi gravado. Princípio: [[Um invariante se garante na estrutura, não no processo]].
- [[Contador de terceiro conta no escopo dele, o seu recorte é delta sobre uma base]] —
  contador que vem de fora acumula por conexão/login, não pela sua unidade de trabalho:
  guarde uma base no marco zero e exponha o delta (taxas recalculadas, listas diffadas);
  valor menor que a base = a fonte zerou. Princípio: [[Balancete é movimento do período, saldo é consequência]].
- [[Polling substitui webhook quando não há IP público]] — quando não dá pra receber
  chamada de fora. Princípio: [[Configuração vem do ambiente, não do código]].
- [[Um stream SSE substitui a constelação de pollings]] — N painéis com `setInterval`
  viram uma conexão SSE de eventos nomeados: push do que é memória (EventEmitter do
  singleton), poll server-side ÚNICO do que é banco/API externa, snapshot no connect.
  Princípio: [[Estado vivo se empurra, não se pergunta]].
- [[Fila de metas pula o item impossível com aviso, e nunca fica parada]] — motor que
  cumpre uma meta e para transforma o dono em agendador; a fila valida de novo na hora
  de executar, pula o inviável com alerta e cai no modo contínuo quando esvazia.
  Princípio: [[Guarde a intenção e o processo se reconstrói dela]].
- [[Num confronto, medir só o seu lado recomenda o alvo que te destrói]] — regra
  simétrica (vantagem de tipo, custo, limite) modelada de um lado só inverte a
  recomendação; a mesma fórmula com os lados trocados vira "quanto você aguenta".
  Princípio: [[Rendimento é vazão vezes tempo em pé, não vazão de pico]].
- [[Bônus multiplicativo só rende onde há folga até o teto]] — bônus percentual sobre
  grandeza que satura rende a distância até o teto, não o tamanho do valor; ordenar
  pelo bruto manda gastar o bônus onde ele se perde. Ordene pelo ganho marginal e
  mostre a fração aproveitada.
  Princípio: [[Rendimento é vazão vezes tempo em pé, não vazão de pico]].
- [[Bônus condicional se avalia contra quem não o recebe]] — bônus que só vale pra parte
  dos candidatos não pode virar filtro: a lista mostra o conjunto inteiro com o bônus
  aplicado onde vale, mais o contrafactual (o mesmo candidato sem ele). "Quem ganha o
  bônus" e "quem paga mais" costumam ser respostas diferentes.
  Princípio: [[Ordene pela grandeza que decide, não pela que impressiona]].
- [[Total acumulado premia a lentidão quando o tempo é livre]] — total é taxa vezes
  duração, então oferecer o acumulado como objetivo escolhe a opção mais lenta. A taxa é
  o objetivo; atividade sem linha de chegada ganha a pergunta invertida (quantidade alvo
  → horas), não a meta emprestada da outra.
  Princípio: [[Ordene pela grandeza que decide, não pela que impressiona]].
- [[Taxa que muda ao longo do trecho se integra, não se amostra na ponta]] — dividir o
  custo do trecho inteiro pela taxa de um ponto cobra tudo naquele ponto, e amostrar a
  ponta erra sempre pro mesmo lado. Some dentro do laço, exiba a média do trecho, e use
  "soma das partes = fórmula fechada" como invariante de conferência.
  Princípio: [[Rendimento é vazão vezes tempo em pé, não vazão de pico]].
- [[Estimativa fraca informa, número verificado ordena]] — número de ranking que soma um
  termo medido com um estimado entrega a ordem ao erro do estimado, e termo que pode ficar
  negativo chega a trocar o sinal. Separe: o verificado ordena, o estimado fica ao lado
  com o limiar que o torna conferível.
  Princípio: [[Ordene pela grandeza que decide, não pela que impressiona]].
- [[Número de regra alheia se lê da fonte, não se congela em constante]] — número que é
  de outro sistema vale pro dia em que foi observado; leia da API em execução com a
  constante de reserva (datada), e a tela lendo a MESMA origem do cálculo — etiqueta
  chumbada é o que faz o valor errado sobreviver à revisão.
  Princípio: [[A definição em dado dirige o comportamento, não um caso no código]].
- [[Quando o campo numérico vem zerado, o número está na frase]] — o campo tipado que
  ninguém preenche (o front do dono renderiza o texto) devolve zero sem erro: frase como
  fonte primária, campo como plano B, casando por proximidade e por palavra inteira, com
  o rótulo cru preservado quando o nome não é reconhecido.
  Princípio: [[Chamada externa tem timeout e erro tratado]].
- [[Estimativa desmentida pela realidade vira veto temporário do motor]] — o fato
  observado (duas quedas no mesmo alvo) veta a recomendação, refaz o plano inteiro e
  caduca na unidade do problema; sem alternativa, alerta em vez de loop calado.
  Princípio: [[Rendimento é vazão vezes tempo em pé, não vazão de pico]].
- [[Estado desejado persistido religa o robô depois do restart]] — a intenção do
  usuário (ligado, modo, alvo) numa tabela; reconexão com backoff renovando o token
  antes de reabrir o WS, e o boot do container relendo a tabela e religando.
  Princípio: [[Guarde a intenção e o processo se reconstrói dela]].
- [[Alerta guarda o retrato do instante; quem tem o id relê a fonte]] — notificação é
  cache com formato datado: campo gravado hoje não existe no alerta de ontem. Guarde o id
  da fonte e releia os visíveis em lote; o retrato fica de fallback pro que sumiu.
  Princípio: [[Estado mutável se lê da fonte no uso, não de cópia guardada]].
- [[Token que rotaciona não tolera cópia longeva, releia do banco antes do uso]] — várias
  frentes renovando o mesmo refresh rotativo: quem guarda cópia em memória apodrece e
  morre em 401 silencioso; toda frente relê o par do banco antes de cada lote REST.
  Princípio: [[Estado mutável se lê da fonte no uso, não de cópia guardada]].
- [[Config que o motor executa mora no servidor e se aplica em todo início de fluxo]] —
  config de automação no localStorage só valia num caminho da UI; salva no servidor com
  `on` embutido, e TODO handler que inicia o fluxo aplica a config salva.
  Princípio: [[Estado mutável se lê da fonte no uso, não de cópia guardada]].
- [[Falha de automação recorrente vira alerta com throttle, não catch vazio]] — loop de
  venda/compra não engole erro: registra evento visível (1 por operação a cada 30min)
  antes de seguir; falha estrutural aparece, transitória não.
  Princípio: [[Chamada externa tem timeout e erro tratado]].
- [[Lote recusado por um item se bissecciona até isolar o culpado]] — API tudo-ou-nada
  (um item inválido = 400 no lote) sem como filtrar antes: bissecção isola os recusados,
  processa o resto e os bloqueia por sessão. Só pra erro determinístico, nunca 5xx.
  Princípio: [[Chamada externa tem timeout e erro tratado]].
- [[Adapter de canal isola o app do provider de mensageria]] — provider externo trocável
  (WhatsApp: Baileys ou Cloud API) fica atrás de uma interface; o app não vê o fornecedor.
- [[Persistir a mensagem não espera a entrega, a entrega é status]] — gravar (durável,
  interno) e entregar (externo, falível) são passos separados; a entrega vira status da
  mensagem (`pendente → enviado → entregue → lida`), nunca pré-requisito da gravação.
- [[Webhook de provedor chega repetido e fora de ordem, a borda tolera os dois]] —
  entrega at-least-once e sem ordem: idempotência numa chave única, status que só avança,
  200 rápido. Princípio: [[Um invariante se garante na estrutura, não no processo]].
- [[Avaliação por comparáveis segmenta a régua por faixa, não mediana única]] — mediana
  única do mercado esmaga a cauda quando o preço não escala linear; segmentar por faixa
  + cadeia de fallback + teto de sanidade pelos anúncios reais.
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

## Retry e diagnóstico

- [[Retry que reusa o cliente queimado esconde o erro da primeira tentativa]] — cliente
  com handshake não sobrevive a uma falha: instância nova a cada volta, e o motivo
  impresso junto da contagem. Senão o laço passa a medir a si mesmo.

## Princípios que mandam aqui

- [[Ordene pela grandeza que decide, não pela que impressiona]]
- [[Permissão se valida no servidor, não na interface]]
- [[Configuração vem do ambiente, não do código]]
- [[Ambiente de dev sobe igual ao de produção]]
- [[Um invariante se garante na estrutura, não no processo]]

---

Voltar para [[Início]]
