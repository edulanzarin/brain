---
tags: [tipo/atomica, camada/padrao, dev/backend]
criado: 2026-09-01
---

# Em canal humano automatizado, o ritmo denuncia antes do volume

> Quando um sistema fala por um canal feito para pessoas — WhatsApp por biblioteca não
> oficial, um portal sem API, um chat de terceiro —, o que identifica automação não é
> quanto ele manda: é **como**. Intervalo exato, rajada em segundos, tráfego às três da
> manhã e conta nova já operando em escala são quatro assinaturas, e nenhuma delas tem
> a ver com o total do dia.

## O problema

O reflexo ao ler "limite de mensagens" é tratar isso como uma cota: cabe tanto por dia,
o resto espera. Mas quem procura robô do outro lado compara o padrão com o de uma
pessoa, e pessoa não manda uma mensagem a cada exatos oito segundos, nem responde
quarenta clientes em dois minutos, nem trabalha em bloco no domingo de madrugada.

A consequência prática é que um sistema pode tomar bloqueio **abaixo** do limite que
julgava respeitar, e é isso que faz o operador concluir que o limite era mentira.

## A costura

Um portão entre "a aplicação quis mandar" e "saiu". A aplicação grava a mensagem como
pendente e enfileira; quem decide o instante é o portão. Cinco travas, e todas moram em
**dado**, não em constante no código, porque cada operação afrouxa ou aperta a sua:

| Trava | Contra o quê |
|---|---|
| rampa por degraus | conta nova operando como conta madura |
| teto do dia | o degrau em que a rampa está hoje |
| teto por minuto | rajada, que é a assinatura mais fácil de ver |
| janela de horário e dias | tráfego fora do expediente de quem ele finge ser |
| pausa sorteada entre mínimo e máximo | cadência exata |

A janela se lê no **fuso da operação**, não em UTC: medida errada, o sistema manda às
cinco da manhã achando que são oito.

E a distinção que mais rende: **responder quem escreveu é muito mais seguro do que
iniciar conversa.** As duas coisas parecem "mandar mensagem" e são atos diferentes aos
olhos de quem vigia. Só a segunda paga a janela inteira.

## O que mais vale lembrar

- **Reconexão também tem ritmo.** Religar em cadência exata é padrão de robô, e várias
  sessões voltando no mesmo instante viram rajada. Sorteio também na volta.
- **Identificação estável.** Trocar a assinatura do cliente a cada subida faz o outro
  lado ver aparelho novo toda vez; string exótica é sinal a mais. Um navegador comum, e
  o mesmo sempre.
- **Nada disso é garantia, e prometer que é faz mal.** Em canal não oficial o bloqueio
  costuma ser definitivo e sem recurso. O portão empurra a probabilidade para baixo;
  quem precisa de garantia troca de canal, e a interface tem que dizer isso ao operador
  na mesma tela em que ele afrouxa o limite.
- Por isso o portão convive com [[Adapter de canal isola o app do provider de mensageria]]:
  a fila é a costura que deixa trocar o meio sem reescrever o sistema.

## Conexões
- Princípio: [[Configuração vem do ambiente, não do código]]
- Irmã: [[Adapter de canal isola o app do provider de mensageria]] · [[Persistir a mensagem não espera a entrega, a entrega é status]] · [[Recusa não é falha: contra o não do servidor, insistir é ruído]]
- Visto em: [[navecrm]]
- Mapa: [[Backend]]
