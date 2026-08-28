---
tags: [tipo/moc]
criado: 2026-07-20
---

# Backend

Técnicas de servidor: Node, rotas, arquivos, integrações. Princípios em [[Base]];
o que é banco de dados tem mapa próprio em [[Dados]].

## Processos e arquivos

- [[Contador de popularidade conta votante, não evento]] — "o mais visto" contando
  evento é placar de F5; a chave (item, dia, votante) transforma spam em `DO NOTHING`, e o
  votante é hash com sal e data — deduplicação, não identificação.
- [[Rotação por período se apura na leitura, e dispensa agendador]] — guarde quando o
  período acaba e apure na primeira leitura depois disso; traz a gravação condicional que
  resolve a corrida da virada.
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
- [[Recorte pequeno em pesquisa anônima identifica, então o painel se recusa a mostrar]]
  — o piso de N respostas por recorte, valendo também para a exportação.
  Princípio: [[Anonimato se perde na saída, não só na entrada]].
- [[Uma resposta canônica de um grupo é um token compartilhado]] — vários podem
  responder, mas só uma resposta vale: um token para o grupo, o primeiro fecha.
  Princípio: [[Um invariante se garante na estrutura, não no processo]].
- [[Permissão composta por papéis somados, não exceção por usuário]] — o cargo
  concentra tudo; a pessoa acumula papéis e o acesso é a união, sem override por
  gente. Princípio: [[Permissão se valida no servidor, não na interface]].
- [[Posse numa permissão binária é duas seções e recorte por linha]] — "dono vê os
  seus, gestor vê todos" não é view/edit: duas seções + recorte por `autor` no
  servidor. Princípio: [[Permissão se valida no servidor, não na interface]].
- [[A ordem da lista de seções é a rota padrão de quem enxerga todas]] — o par por
  posse quebra só para o admin, que enxerga as duas: a home entrega a primeira da
  lista, então a de gestão vem antes. Princípio:
  [[A home de um módulo é o resumo que carrega sozinho; automação não abre sozinha]].
- [[Dado externo sem par no cadastro local não tem escopo]] — o que a integração
  não casou não tem chave para recortar; `null` ali é "não sei de quem é", não
  "de todos". Princípio: [[Permissão se valida no servidor, não na interface]].
- [[Quando a API cobra uma chamada por item, filtrar não economiza]] — se o
  recurso exige id no caminho, o custo é N chamadas com filtro ou sem; traga
  tudo e recorte na leitura. Princípio:
  [[Fator que domina o resultado não entra na conta por estimativa]].
- [[Parâmetro de presença perde o efeito se você der um valor a ele]] — `?config`
  e `?config=1` não são a mesma coisa; o construtor de query sempre escreve
  `chave=valor` e a resposta volta vazia sem erro. Princípio:
  [[Contador que conta sucesso de promessa afirma que deu certo]].
- [[Processo longo prova que está vivo batendo, não deixando a linha aberta]] —
  linha sem fim também é o que o processo morto deixa; presença se prova por
  carimbo recente, e deploy deixa de travar o módulo. Princípio:
  [[Contador que conta sucesso de promessa afirma que deu certo]].
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
- [[Diff de catálogo externo carimba a versão do extrator]] — fonte que muda sem changelog
  vira changelog automático comparando o snapshot com o download de agora; o snapshot
  guarda o número da pipeline que o produziu, e lados de pipeline diferente não se
  comparam.
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

- [[Provedor de pagamento entra por interface, e o simulado é a primeira implementação]] —
  o sistema fala com um contrato de três campos, não com o SDK do gateway; o provedor
  simulado deixa o fluxo de venda existir inteiro antes de existir credencial. Princípio:
  [[Chamada externa tem timeout e erro tratado]].
- [[Recusa não é falha: contra o não do servidor, insistir é ruído]] — retry serve pra
  falha passageira; contra recusa (banido, sem permissão, cota) insistir só gasta
  tentativa e esconde o motivo. Classifique 403/401/429 antes de decidir o retry, guarde
  a frase crua do outro lado e desligue a INTENÇÃO, não só a conexão. Princípio:
  [[Chamada externa tem timeout e erro tratado]].
- [[Duas listas parecidas respondem perguntas diferentes, e a errada some com o item]] —
  catálogo e posse trazem os mesmos campos e divergem só na ausência; usar um pelo outro
  faz o item exclusivo de uma das listas sumir sem erro nenhum.
  Princípio: [[Estado mutável se lê da fonte no uso, não de cópia guardada]].
- [[403 do escudo não é 403 do dono da API]] — a borda (Cloudflare, WAF) responde com os
  mesmos status da aplicação, e o `403` dela fala do seu PEDIDO, não da sua conta. Lido
  como recusa de conta, vira estado terminal por um desafio de trinta segundos. A
  assinatura é o corpo: API responde JSON, escudo responde HTML.
  Princípio: [[Laço que trata toda falha igual apaga a causa da primeira]].
- [[Re-chavear um sistema é refactor mudo, force o compilador a achar as chamadas]] —
  trocar por quem o sistema é chaveado não muda um tipo: os dois ids são `string`, e cada
  chamada segue compilando com a chave errada. Mude a aridade ou troque dois parâmetros
  por um objeto, e o erro de tipo enumera o trabalho.
  Princípio: [[Um invariante se garante na estrutura, não no processo]].
- [[Campo cujo nome você não sabe se lê do payload, nunca se chuta]] — API tolerante
  ignora chave desconhecida e devolve 200, então o palpite errado não falha, PASSA: o
  seletor mostra a escolha aplicada e o comportamento nunca muda. Leia o nome do campo no
  objeto que a fonte já devolve, e escreva de volta na mesma chave.
  Princípio: [[Peça o que a fonte mostra, não o que você precisa]].
- [[Limiar conta a unidade que se consome, não o balde que a contém]] — reposição
  automática compara estoque com piso, e o estoque costuma ser soma de categoria enquanto
  o consumo é de UM item: com a soma alta o piso nunca dispara, e a coisa que acabou
  continua acabada. O sintoma é inércia, não erro.
  Princípio: [[Espelhar por balde esconde item no lugar errado]].
- [[Adquirir o recurso exclusivo é uma ação, usá-lo é outra]] — quando "ligar" faz junto
  a primeira tarefa, essa tarefa vira pré-requisito de todas as outras e trocar de tarefa
  passa por largar o recurso. Dois estados: quero segurando, e qual tarefa.
  Princípio: [[Um invariante se garante na estrutura, não no processo]].
- [[Objetivo é exclusivo, interruptor é combinável]] — duas automações que escrevem no mesmo valor não são duas chaves, são uma escolha; com N booleanos o estado impossível existe e alguém liga.
  Princípio: [[Um invariante se garante na estrutura, não no processo]].
- [[Freio de oscilação vale para a máquina, não para a ordem de quem manda]] — o piso de tempo entre decisões automáticas, aplicado largo, freia também o clique: some com a resposta e não deixa rastro, porque do ponto de vista dele nada aconteceu.
  Princípio: [[Estado mutável se lê da fonte no uso, não de cópia guardada]].
- [[O código com que o socket fecha é a classificação que o retry precisa]] — a faixa
  4000–4999 do WebSocket é da aplicação, e é ali que o outro lado diz se foi credencial,
  roteamento ou recusa. Descartar o número troca o diagnóstico por um laço infinito.
  Princípio: [[Laço que trata toda falha igual apaga a causa da primeira]].
- [[Socket que não abre não emite evento, e só um temporizador percebe]] — handshake que
  não completa não dispara `open`, `close` nem `error`: nada falha formalmente e a
  reconexão nunca é agendada. O sintoma é um "conectando" que não termina.
  Princípio: [[Laço que trata toda falha igual apaga a causa da primeira]].
- [[Comando sem resposta precisa de vigia, não de fé]] — frame que o servidor aceita ou ignora, quando se perde, não gera erro: gera silêncio com cara de espera. Observe o efeito que ele deveria produzir e reaja à ausência dele.
  Princípio: [[Laço que trata toda falha igual apaga a causa da primeira]].
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
- [[Duração conta o dia inicial, prazo para agir conta do dia seguinte]] — duração é
  intervalo fechado (`início + n − 1`), prazo para agir é deslocamento (`início + n`);
  a mesma palavra "45 dias" cobre as duas e o erro sai como data plausível um dia
  adiante. Regra numa função só, exemplo conferível no comentário, e migration para a
  data derivada que já ficou gravada.
  Princípio: folha isolada.
- [[Conta da natureza de serviço vem do hábito, não da tabela do ERP]] — a tabela de
  contabilização do serviço aponta pra conta aposentada (acerta 62%; o hábito da
  natureza acerta 86%), e a natureza genérica não tem conta nenhuma pra cobrar.
  Aprende a moda por (empresa, filial, natureza) com dominância de 80%, marca a
  linha sem hábito como conta variável e espelha por NOTA, não por conta.
  Princípio: [[Config declarada envelhece; quem diz a regra é o comportamento observado]].
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
- [[Casar o favorecido do extrato com a folha - CPF prova, nome indicia]] — o mascarado do
  PIX esconde as pontas e mostra o miolo, que é o que discrimina; nome só casa acompanhado
  de sobrenome, e quando casa acompanha a decisão em vez de tomá-la.
- [[De-para determinístico com override que vira aprendizado]] — casar conta/código
  de dois sistemas por cascata (chave → estrutura → descrição restrita), sem IA; a
  correção do humano vira override salvo e o de-para melhora a cada uso. Princípio:
  [[A definição em dado dirige o comportamento, não um caso no código]].

## Cabeçalhos HTTP

- [[CSP só aparece no build de produção, toda origem externa vai no allowlist]] —
  o dev server não tem CSP; recurso externo (fonte, tile) só quebra em prod.
  Princípio: [[Verificar no build de produção, não só em dev]].

## Retry e diagnóstico

- [[Processo que segura sessão viva não morre em exceção não tratada]] — crash-only é certo
  para servidor sem estado e errado para quem segura socket por usuário; quem decide é o
  que se perde na queda. Princípio:
  [[Rendimento é vazão vezes tempo em pé, não vazão de pico]].
- [[Retry que reusa o cliente queimado esconde o erro da primeira tentativa]] — cliente
  com handshake não sobrevive a uma falha: instância nova a cada volta, e o motivo
  impresso junto da contagem. Senão o laço passa a medir a si mesmo.

## Engenharia reversa de sistema fechado

- [[A interface do sistema explica o que a API dele esconde]] — a tela precisa se explicar
  a um humano, então ela NOMEIA o mecanismo que o JSON só numera: um print da ficha do boss
  deu o que `mult` multiplica, como a força se calcula e o elemento que faltava. E expôs o
  arquivo público desatualizado, que listava outras recompensas. Depois expôs o pior dos
  três furos — o campo público que MENTE com valor bem tipado e plausível — e a regra que
  saiu daí: onde há botão desligado, há requisito publicado. Peça o print.
- [[Tirar o dado errado não põe a verdade no lugar]] — o outro lado do mesmo trabalho:
  achado o dado falso e cortado, confira o que a AUSÊNCIA dele passa a afirmar.

## Referência de sistema externo

- [[Poke Idle World - endpoints publicos de dados]] — o catálogo inteiro em JSON sem auth.
- [[Poke Idle World - regras de breeding]] — regras curadas, que não vêm em endpoint nenhum.
- [[Poke Idle World - evolucao e a troca do Eevee]] — os dois caminhos de evolução, e o
  Eevee, que não evolui: é trocado com um NPC por um de cinco, cada um pedindo a sua pedra.
- [[Poke Idle World - TM e os discos]] — o maior salto de poder do jogo (60 contra 43,3 do
  melhor natural), quem aprende cada disco e as três armadilhas do catálogo.

- [[Recurso indivisível se aloca pelo salto, não pelo resultado]] — com UM item pra dar a
  vários candidatos, ordene pelo quanto ele MUDA cada um e não pelo estado em que cada um
  fica: quem termina no topo costuma ser quem já estava perto dele.

## Princípios que mandam aqui

- [[Ordene pela grandeza que decide, não pela que impressiona]]
- [[Permissão se valida no servidor, não na interface]]
- [[Configuração vem do ambiente, não do código]]
- [[Ambiente de dev sobe igual ao de produção]]
- [[Um invariante se garante na estrutura, não no processo]]
- [[Índice só é identidade enquanto a coleção não muda]]

- [[Multiplicador de contexto entra depois da razão, não dentro dela]] — o que entra
  numa razão `a/(a+b)` é a grandeza crua dos DOIS lados; bônus de contexto multiplica o
  resultado. Inflar um prato faz a razão saturar e apagar a diferença que ela media.

---

Voltar para [[Início]]
