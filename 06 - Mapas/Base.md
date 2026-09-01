---
tags: [tipo/moc]
criado: 2026-07-20
---

# Base

Os **princípios**: regras que não dependem de framework, de linguagem nem de projeto.
É a camada mais estável e a primeira a consultar quando aparece um problema novo —
quase sempre já existe um princípio que cobre o caso, e o trabalho vira só escrever a
técnica nova embaixo dele.

Meta-regras: [[Camadas do conhecimento - princípio, padrão, aplicação]] ·
[[Conhecimento pertence à base, não ao projeto]]

## Design — como a tela se organiza

Independe de CSS, de Tailwind e de React. Técnicas concretas em [[Design]].

- [[Token semântico em vez de valor literal]] — nenhum valor cru no componente.
- [[Escala fechada em vez de valor solto]] — espaço, texto e raio saem de conjunto fixo.
- [[Container tem largura máxima e respiro constante]] — as três medidas de todo container.
- [[Hierarquia por superfície, não por borda]] — profundidade por fundo; borda é último recurso.
- [[Todo estado da tela tem visual]] — carregando, vazio, erro e sucesso são design.
- [[A casca se compartilha por público, não por marca]] — mesma empresa e mesmo logo não fazem uma casca só; o que faz é o mesmo público no mesmo ritmo de volta. Duas coisas lidas em ritmos diferentes, empilhadas na mesma página, ensinam qual é a principal — e ensinam errado.
- [[Catálogo de componentes é contrato vivo, não documentação]] — a peça entra no catálogo no commit em que nasce, com o porquê junto; o que fica escondido atrás de um clique se mostra também parado.
- [[Tela que abre vazia tem que ensinar, tela que abre cheia não]] — catálogo se explica de olhar; ferramenta pede número que mora fora dela e precisa de manual.
- [[O que responde pergunta rara não ocupa a rolagem de todo mundo]] — a altura da página é orçamento pago por toda visita; bloco que interessa a poucos vai pro modal, com o gatilho carregando o estado que decide se vale abrir.
- [[Nota carrega só o que a pessoa não sabe]] — legenda diz de onde o número saiu e o que ele não conta; repetir o campo é ruído com cara de ajuda.
- [[Texto de interface soa a IA pelo ritmo, não pelo assunto]] — travessão emendando, "não é X, é Y" e a explicação que ninguém pediu: quem lê não aponta o erro, só diz que parece gerado, e a desconfiança passa do texto pro número ao lado.
- [[Custo de processo aleatório se orça pela cauda, não pela média]] — estimativa de quantas tentativas mostra melhor caso, típico e azarado; média sozinha é armadilha de orçamento.
- [[Peça o que a fonte mostra, não o que você precisa]] — campo que a pessoa não tem de onde copiar está errado; peça as grandezas visíveis e derive, e leve a incerteza da derivação adiante.
- [[A régua sai da distribuição, não dos extremos]] — quem decide a escala de barra, trilho ou eixo é a forma do dado, não o seu máximo.
- [[A tela não afirma mais precisão do que a fonte tem]] — casa decimal é o tamanho da afirmação; zero arredondado e ponto onde só há faixa inventam exatidão.
- [[Travar o valor não impede a tela de afirmar a partir dele]] — `clamp` conserta o que se lê, não o que se conclui: projeção, cor, conselho e agregado continuam saindo do valor travado.
- [[Limiar em grandeza contínua vira degrau, e o degrau decide a ordem]] — `if x >= K` que responde número, e não rótulo, faz 0,1% de entrada virar 2x de saída; o teste é medir a razão entre vizinhos.
- [[Estado compartilhável mora na URL]] — o que descreve a vista vai pro link.
- [[Estado de tela pertence à seção, não à página]] — estado no menor escopo que resolve.
- [[Entidade auxiliar se cria no ponto de uso, não em tela própria]] — grupo/etiqueta é campo, não aba.

## Infraestrutura — como um projeto sobe

Independe de Docker. Técnicas concretas em [[Infra]].

- [[Configuração vem do ambiente, não do código]] — a raiz de todas as outras.
- [[O nome do projeto governa o nome dos recursos]] — `prospects-app`, `prospects-db`.
- [[Porta interna é constante, porta externa é configuração]] — porta se escolhe na chamada.
- [[Uma faixa de portas por projeto]] — o mapa de portas reservadas.
- [[Ambiente de dev sobe igual ao de produção]] — mesma receita nos dois lugares.

## Dados e reconciliação — comparar dois mundos

Independe de banco e de domínio (vale contábil, estoque, qualquer conferência).

- [[Balancete é movimento do período, saldo é consequência]] — compare movimento, não saldo.
- [[Espelhar por balde esconde item no lugar errado]] — a granularidade do espelho decide o que a reconciliação enxerga.
- [[Sobre fonte read-only, o editável mora no seu banco chaveado pela identidade dela]] — a fonte que você não pode escrever fica intacta; correção, inclusão e renomeação moram do seu lado, o merge é seu.
- [[Config declarada envelhece; quem diz a regra é o comportamento observado]] — config é afirmação feita uma vez, comportamento é afirmação feita a cada lançamento; quando todas as ocorrências da mesma chave desviam pro mesmo lugar, quem está errado é a régua, e o conferidor passa a cobrar o hábito (com dominância, não maioria simples).
- [[Onde não há regra, espelhar é mais honesto que arbitrar]] — item sem regra copia o real pro esperado e fecha em zero; esperado inventado gera diferença que fala do conferidor, não do mundo. A classificação é por item, nunca por balde.
- [[A unidade de contagem é o ato, não a linha que ele deixou]] — um gesto grava as linhas que o modelo exigir (uma por funcionário, uma por imposto, uma por recarga), e contar linha mede o formato da tabela, não o trabalho. Medido: 18.504 linhas para 137 atos. A distorção REORDENA o ranking, não só infla o total; contagem idêntica entre categorias é a assinatura do lote.
- [[Diferença entre duas leituras só fala do mundo se o instrumento não mudou]] — o diff entre duas fotos soma o que mudou lá fora com o que mudou em quem fotografou; carimbe a versão do extrator no dado e recuse comparar através dela. Patch faltando é buraco, patch inventado é mentira.

## Sistemas vivos — estado, canal e processo

Vale pra qualquer coisa que roda sozinha (robô, worker, sessão longa) e pra qualquer
UI que mostra dado que muda sem o usuário pedir.

- [[Estado vivo se empurra, não se pergunta]] — quem produz o estado avisa; a interface assina um canal. Polling é exceção com motivo.
- [[Guarde a intenção e o processo se reconstrói dela]] — o que o usuário quer se persiste separado do que está acontecendo; restart e queda não apagam decisão.
- [[Estado mutável se lê da fonte no uso, não de cópia guardada]] — o que outra frente escreve/rotaciona (credencial, config) se relê da fonte persistida na hora de usar; cópia longeva apodrece em silêncio.

## Segurança

- [[Permissão se valida no servidor, não na interface]] — esconder botão não é segurança.
- [[A assinatura autentica o dado, não quem o trouxe]] — confie na assinatura (HMAC), não no canal; vale pro webhook externo e pro cookie de sessão próprio.
- [[Anonimato se perde na saída, não só na entrada]] — não guardar identidade é a metade fácil; agregado com recorte fino reconstrói a pessoa, então recorte abaixo de N não se mostra nem se exporta.

## Ofício — como eu trabalho

- [[Verificar no build de produção, não só em dev]]
- [[Semear teste cria linha nova, não muta linha real]] — e a limpeza vai no `finally`, senão o teste que quebra no meio deixa lixo que o próximo encontra e não entende.
- [[A regra mora fora da porta que a chama]] — regra escrita dentro do formulário/rota só existe quando a porta existe, e só se confere atravessando a porta. Quando o teste morre em `cookies` ou `revalidatePath`, a resposta não é simular o framework: é tirar a regra de dentro dele.
- [[Dado escrito por dois caminhos precisa de uma regra só, fora dos dois]] — o cadastro cria e a tela corrige; com a regra copiada nos dois, o lado escrito depois passa a aceitar o que o outro recusa, e nada dá erro. A duplicação nasce na hora da segunda tela.
- [[Recurso sem escrita parece pronto quando a semente preenche a leitura]] — a metade que mostra é a que se vê e a que a semente satisfaz sozinha; a conferência de que algo existe é apontar a função que cria a linha, nunca a tela que a lê.
- [[Alternar é uma ação só, porque quem sabe o estado é o banco]] — guardar e tirar não são duas funções: quem escolhe o sentido é o que estava gravado, e a decisão sai de uma escrita atômica, não de um ler-e-depois-escrever.
- [[O acordo congela na linha, a política vale do próximo em diante]] — preço, percentual e alíquota moram no cadastro porque precisam mudar, e é por isso que o que já foi combinado não pode continuar apontando para eles. O teste: mudar o número muda algum relatório do mês passado?
- [[Migração de dados mantém o antigo como reserva até a virada]]
- [[A definição em dado dirige o comportamento, não um caso no código]] — o que varia por um eixo conhecido vira dado que uma peça lê.
- [[Um invariante se garante na estrutura, não no processo]] — "no máximo uma X" recusa-se no schema, não na expectativa do fluxo.
- [[O que tem ciclo de vida próprio é entidade própria, não modo de outra]] — quantas vezes acontece, se repete e o que a fecha: divergiu da tabela hospedeira, é entidade própria, não flag dela.
- [[Recorrência guarda a receita e o próximo disparo, não N ocorrências futuras]] — periódico é receita + ponteiro; o job materializa uma por vez.
- [[Plataforma de IA hospedada prende o app pelo banco]]
- [[Progresso idle é função pura do tempo semeada, não simulação tick a tick]] — o ganho de um período se resolve com uma função determinística sobre o Δt, não simulando o relógio; offline honesto e à prova de cliente.
- [[Versão é corte deliberado em SemVer, não efeito de cada merge]]
- [[Coerência em geração vem de âncora, não de liberdade]] — gerar coerente (procedural ou IA) vem de prender a uma referência/peça curada, não de gerar livre.
- [[Rendimento é vazão vezes tempo em pé, não vazão de pico]] — ao ranquear opções, o custo de falhar entra como tempo parado dentro do mesmo número; vazão de pico escolhe o que quebra primeiro.
- [[Ordene pela grandeza que decide, não pela que impressiona]] — a métrica intuitiva costuma ser uma parcela da grandeza real; a dimensão esquecida é tempo, multiplicação ou folga até o teto, e errar por ela erra sempre pro pior lado.
- [[Laço que trata toda falha igual apaga a causa da primeira]] — repetir até dar certo apaga o diagnóstico: a primeira tentativa era a única que carregava a causa, e o erro final costuma ser do próprio laço. Classifique antes de repetir, e nunca reuse recurso queimado.
- [[Contador que conta sucesso de promessa afirma que deu certo]] — desistir cedo também resolve a promessa, então "N/N ok" some com quem foi pulado. Conte por desfecho, e desconfie da métrica escrita junto com o código.

- [[Ver o plano e mandar executar são duas ações]] — quando escolher o modo já liga o modo, quem ainda está decidindo se confia precisa delegar para poder olhar, e depois desfazer. O modo é do olho, o objetivo é da máquina.
- [[Fórmula verificada só vale na escala em que foi verificada]] — a conferência valida a fórmula E a procedência dos números; trocada a fonte, a aritmética não reclama de unidade e o erro sai formatado. Cheque contra um invariante que a fonte declare.
- [[Casar dado do mundo real é por classe de equivalência, não por igualdade]] — telefone, nome e documento chegam em várias grafias corretas; comparar string diz "não achei" onde havia. Estreito demais duplica, largo demais funde — e quando a classe cabe mais de uma pessoa, o casamento virou indício e precisa entregar a dúvida.
- [[Ausência de leitura cai no valor que dispara a ação]] — leitura que falhou vira `0`, `[]` ou `null`, e num sistema que decide por limiar esses são exatamente os valores do lado "faltando, faça alguma coisa". O bug não dá número errado: dá ação máxima, a cada ciclo, em silêncio. Não saber é razão pra não agir.
- [[Produtividade se mede pela hora do registro, não pela data do fato]] — todo dado de trabalho tem duas datas (o fato e o carimbo de quem registrou); medir gente pela primeira faz o mês fechado mudar de número depois de avaliado.
- [[Ausência só aparece contra o universo, nunca contra a tabela de eventos]] — quem não trabalhou não gera linha, então o placar feito pra medir atendimento é justamente o cego pro buraco de atendimento; o denominador vem do cadastro, e o recorte de quem deveria estar nele vem do comportamento.
- [[Razão só afirma quando os dois lados vêm do mesmo trabalho]] — dividir dois números que a tela tem à mão é aritmética válida afirmando fato falso; volume de máquina sobre hora de gente mede a máquina e cobra da gente, e recorte que muda um lado só troca de indicador, não de fórmula.
- [[Reduzir a cardinalidade vem antes de enriquecer]] — sobre fato grande, corte o volume primeiro e só então junte nome, conte distinto e fatie; trabalho caro antes do corte é trabalho feito milhões de vezes à toa.
- [[Índice só é identidade enquanto a coleção não muda]] — posição só identifica enquanto o array é o mesmo; filtrou, compactou ou reordenou, o índice guardado aponta pra outra coisa e a leitura continua válida, então nada avisa.
- [[Identificador que já circulou não é mais seu para mudar]] — endereço que nasce de campo editável publica um novo e mata o antigo a cada edição, e quem paga é quem tinha o link. Separe o que se lê do que se endereça; se a troca precisa existir, ela tem trava e redirecionamento.
- [[Fator que domina o resultado não entra na conta por estimativa]] — termo desconhecido grande o bastante pra mandar sozinho no resultado fica FORA da conta e DENTRO da tela; chutado, o número deixa de falar do modelo e passa a falar do chute.
- [[Tirar o dado errado não põe a verdade no lugar]] — apagar um valor falso não deixa a tela em silêncio, deixa ela no caminho PADRÃO, e o padrão também afirma; metade das vezes ele afirma o oposto, que é igualmente mentira e agora sem nada errado no código pra procurar.
## Cérebro — como este vault funciona

- [[Camadas do conhecimento - princípio, padrão, aplicação]]
- [[Conhecimento pertence à base, não ao projeto]]

## Como esta camada cresce

Princípio novo entra quando o **mesmo aprendizado aparece pela segunda vez**, em
contexto diferente. Antes disso é técnica, e técnica mora em [[Design]], [[Infra]],
[[Frontend]], [[Backend]] ou [[Dados]].

Princípio que nunca teve dois casos concretos costuma ser regra inventada, não regra
aprendida — e regra inventada é o que faz base virar entulho.

- [[Calibre nas pontas, o meio esconde o defeito]] — o caso médio fecha por acaso; o
  defeito mora no extremo, e o extremo de baixo é onde o usuário novo entra.
- [[Desgaste constante contra recuperação constante é bimodal]] — mexer nas taxas move a
  fronteira, nunca cria a faixa do meio; ela vem de um consumível que responde ao estado.

---

Voltar para [[Início]]
