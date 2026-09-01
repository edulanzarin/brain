---
tags: [tipo/atomica, camada/padrao, design]
criado: 2026-09-01
---

# Fato vai em selo, estado vivo vai no retrato

> Numa fileira de selos, misturar o que **não muda** ("documento conferido", "verificado") com o que **muda no meio da tarde** ("em expediente", "online") faz a fileira inteira parecer volátil — e enterra, no meio de iguais, justamente o dado que a pessoa quer de relance. Fato fica escrito num selo; estado vivo vai para o canto do retrato, que é onde toda rede social já ensinou a olhar.

## O corte

A pergunta que separa os dois: **isso ainda vale amanhã?**

- Vale → selo. Conferência, categoria, plano contratado.
- Não vale → indicador de presença, junto da identidade (retrato, nome), não na
  lista de atributos.

O ganho não é só estético. Quem lê uma fileira de cinco selos lê cinco coisas do
mesmo peso; quem vê uma bolinha verde no retrato lê uma coisa antes de ler
qualquer outra — que é exatamente a prioridade certa quando o estado decide se
vale a pena continuar.

## A cor do "não": cinza, não vermelho

Vermelho é a escolha instintiva para "não está" e costuma ser a errada:

1. **Vermelho já quer dizer outra coisa no seu sistema.** Se existe um tom de
   perigo (registro suspenso, rejeitado, erro), o mesmo vermelho em dois lugares
   passa a dizer duas coisas, e o leitor não sabe qual.
2. **Em retrato, vermelho é o cânone de "ocupada"**, não de "fora" — quem vem de
   Slack ou Teams lê "não me incomode", que é outro estado.
3. **Verde/vermelho é o pior par possível** para o daltonismo mais comum.

Verde cheio contra **cinza apagado** separa por *luminância*, então continua
legível em preto e branco e para quem não distingue matiz. É a mesma família de
[[Glifo miúdo é lido como o símbolo mais próximo que a pessoa já conhece]]: o
cânone que a pessoa já traz vence a intenção de quem desenhou.

## Ausência de dado não é o estado "não"

Se a pessoa nunca declarou horário, a bolinha **some** — não fica cinza. Cinza
afirmaria "fora do expediente" sobre quem nunca disse ter expediente, que é o
sistema inventando um fato. Três estados, não dois: sim, não, e não sei.

## A bolinha nunca fala sozinha

Cor não é rótulo. O indicador leva `title` para o ponteiro e texto de leitor de
tela — senão o dado existe só para quem enxerga bem e conhece a convenção.

## Quando o canto já está ocupado

O canto inferior direito é disputado: presença, marca de "tem story", selo de
verificação. Quem cede é **quem só reforça**, não quem informa. Se o anel do
retrato já diz que ele abre alguma coisa, a marca de story é reforço e pode
subir; a presença fica com o lugar canônico, porque é a única ali que carrega
informação nova.

## Conexões
- Princípio: [[Ordene pela grandeza que decide, não pela que impressiona]]
- Irmã: [[Chip que serve a duas grandezas declara qual delas mostra]] · [[Título e metadados no mesmo flex-wrap deixam o dado decidir a quebra]]
- Visto em: [[Privello]] — cabeçalho do perfil
- Mapa: [[Design]]
