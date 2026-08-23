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
- [[Tela que abre vazia tem que ensinar, tela que abre cheia não]] — catálogo se explica de olhar; ferramenta pede número que mora fora dela e precisa de manual.
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

## Sistemas vivos — estado, canal e processo

Vale pra qualquer coisa que roda sozinha (robô, worker, sessão longa) e pra qualquer
UI que mostra dado que muda sem o usuário pedir.

- [[Estado vivo se empurra, não se pergunta]] — quem produz o estado avisa; a interface assina um canal. Polling é exceção com motivo.
- [[Guarde a intenção e o processo se reconstrói dela]] — o que o usuário quer se persiste separado do que está acontecendo; restart e queda não apagam decisão.
- [[Estado mutável se lê da fonte no uso, não de cópia guardada]] — o que outra frente escreve/rotaciona (credencial, config) se relê da fonte persistida na hora de usar; cópia longeva apodrece em silêncio.

## Segurança

- [[Permissão se valida no servidor, não na interface]] — esconder botão não é segurança.
- [[A assinatura autentica o dado, não quem o trouxe]] — confie na assinatura (HMAC), não no canal; vale pro webhook externo e pro cookie de sessão próprio.

## Ofício — como eu trabalho

- [[Verificar no build de produção, não só em dev]]
- [[Semear teste cria linha nova, não muta linha real]]
- [[Migração de dados mantém o antigo como reserva até a virada]]
- [[A definição em dado dirige o comportamento, não um caso no código]] — o que varia por um eixo conhecido vira dado que uma peça lê.
- [[Um invariante se garante na estrutura, não no processo]] — "no máximo uma X" recusa-se no schema, não na expectativa do fluxo.
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
## Cérebro — como este vault funciona

- [[Camadas do conhecimento - princípio, padrão, aplicação]]
- [[Conhecimento pertence à base, não ao projeto]]

## Como esta camada cresce

Princípio novo entra quando o **mesmo aprendizado aparece pela segunda vez**, em
contexto diferente. Antes disso é técnica, e técnica mora em [[Design]], [[Infra]],
[[Frontend]], [[Backend]] ou [[Dados]].

Princípio que nunca teve dois casos concretos costuma ser regra inventada, não regra
aprendida — e regra inventada é o que faz base virar entulho.

---

Voltar para [[Início]]
