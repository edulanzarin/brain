---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-19
---

# Comando que responde ok e não muda nada tem pré-condição de estado

> Mutação que o servidor aceita, confirma e não produz efeito quase nunca é bug de envio.
> É pré-condição: o comando existe para outro estado do sistema. Insistir nele é loop —
> a automação precisa **sair do estado, agir e voltar** pela intenção que guardou.

## O sintoma

O comando vai, o ack volta, o estado fica igual. Como o ack chegou, o código conclui que
funcionou e repete no próximo ciclo — e o loop se fecha em silêncio: nada quebra, nada
alerta, só não anda. É o parente sonoro de
[[Confirme a mutação pelo estado que ela deixa, não pelo ack que pode não chegar]]:
lá o ack não vinha; aqui ele vem e mente.

## Onde procurar a pré-condição

No **cliente oficial**, que já a conhece — ele desabilita o botão, mostra o aviso, ou
simplesmente não oferece a ação naquela tela. Duas leituras costumam resolver:

- **De onde a ação é disparada.** Um comando oferecido por um NPC da cidade não vale
  dentro da masmorra, mesmo que o frame não tenha nenhum campo de lugar.
- **Que texto o cliente mostra quando recusa.** A mensagem de bloqueio nomeia a
  pré-condição e costuma listar as saídas alternativas.

O caminho é o de [[O bundle público do cliente entrega o contrato da API sem documentação]],
com uma diferença: aqui não se procura o endpoint, e sim **a condição** em que ele age.

## O desenho da recuperação

Estado bloqueante é desvio, não fim de linha. A automação:

1. **guarda a intenção** (o que estava fazendo antes de travar);
2. **sai do estado** que bloqueia — e sair costuma ser um comando explícito, não um
   efeito colateral;
3. **age** onde a ação vale;
4. **confirma pelo estado** e **volta** para a intenção guardada, sozinha.

Vale a **saída rápida** quando ela existir e custar pouco: se dá pra resolver sem sair
(um item na bolsa, um retry local), tente isso primeiro e deixe o desvio como plano B,
com prazo. Sem prazo, o plano B nunca roda; sem plano B, o loop volta.

E dê **cara** ao estado na interface: enquanto se recupera, a automação não está
trabalhando. Continuar mostrando "ao vivo" é o que faz o dono achar que está tudo bem.

## Visto em

No piwdex, o robô mandava `joy-heal` com o pokémon desmaiado e ele seguia morto: a
enfermeira é NPC da cidade e não alcança quem está EM CAMPO — e o jogo ainda recusa
entrar em caçada com o líder caído, então até a troca de hunt batia na parede. Ordem
nova: `field-revive` (gasta um Revive da bolsa e levanta sem sair), senão `leave-hunt` +
`joy-heal` (grátis, fora do campo) e volta pro mesmo slug quando o HP enche.

## Conexões
- Princípio: [[Guarde a intenção e o processo se reconstrói dela]]
- Irmã: [[Confirme a mutação pelo estado que ela deixa, não pelo ack que pode não chegar]]
- Depende de: [[O bundle público do cliente entrega o contrato da API sem documentação]]
- Parente: [[Falha de automação recorrente vira alerta com throttle, não catch vazio]]
- Visto em: [[piwdex]]
- Mapa: [[Backend]]
