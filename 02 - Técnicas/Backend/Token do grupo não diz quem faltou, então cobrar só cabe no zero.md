---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-09-21
---

# Token do grupo não diz quem faltou, então cobrar só cabe no zero

> A credencial compartilhada por um grupo identifica o **grupo**, não a pessoa. Se
> quem responde se identifica por texto livre, o sistema nunca sabe QUEM falta — e
> "mandar lembrete pra quem não respondeu" só tem resposta honesta enquanto o
> contador está em zero. Com uma resposta que seja, cobrar de novo bate também em
> quem respondeu.

## O problema

O link é do setor ([[Uma resposta canônica de um grupo é um token compartilhado]]):
os N gestores recebem o mesmo endereço e cada um responde a sua, digitando o próprio
nome. Duas semanas depois metade não respondeu e vem o pedido óbvio: "cobra quem
faltou".

Parece trivial e não é. A lista de quem **deveria** responder existe (os gestores
ativos do setor); o que não existe é o casamento entre ela e as respostas. O nome
digitado é texto livre — "Ana", "ana paula", "Coordenação" —, o e-mail é opcional, e
um mesmo link não distingue quem o abriu. Sem esse casamento não dá para subtrair um
conjunto do outro ([[Ausência só aparece contra o universo, nunca contra a tabela de eventos]]):
o universo está lá, os eventos não sabem apontar para ele.

## A solução

Reconhecer que a granularidade da credencial já decidiu a granularidade da cobrança, e
escolher entre três desenhos — no projeto, não no dia em que pedirem o lembrete:

1. **Cobrar só no zero.** O lembrete existe só para a avaliação que ninguém respondeu.
   É uma condição, não uma limitação envergonhada: escreve-se como predicado e vale
   igual na linha e no lote.
2. **Um token por pessoa.** Rastreia perfeito e devolve o problema que o token de grupo
   resolvia: N respostas concorrentes sobre o mesmo caso, sem dizer qual vale.
3. **Identidade escolhida de uma lista.** Quem responde se seleciona entre os gestores
   cadastrados em vez de digitar. Mantém o link único e permite cobrar quem falta — ao
   preço de o cadastro ter de estar certo, e de a saída "outro" reabrir o buraco.

```sql
-- o predicado do (1): saiu, segue aberta, e ninguém respondeu
select count(*) from avaliacao a
 where a.rodada_id = $1 and a.status = 'enviado' and a.encerrado_em is null
   and not exists (select 1 from resposta r where r.avaliacao_id = a.id)
```

## O que mais vale lembrar

- **O mesmo predicado serve ao botão e ao número dele.** A tela precisa dizer "cobrar
  quem não respondeu (7)" antes de o RH clicar; se a contagem vier de outro lugar que
  não o filtro do disparo, um dia diverge e o botão promete o que não faz.
- **Cobrança quer rastro em linha, não coluna.** `ultimo_lembrete` responde "quando",
  mas a pergunta real é a insistência: quantas vezes, quando e para quem. Uma linha por
  cobrança responde as três e não perde as anteriores.
- **O e-mail de cobrança assume que fala com quem já respondeu.** Como o disparo não
  sabe distinguir, o rodapé precisa carregar o "se você já respondeu, desconsidere" —
  é a honestidade do desenho aparecendo no texto.
- **Falha de um alvo não derruba o lote.** Setor sem gestor ou SMTP fora do ar viram
  nome numa lista de falhas e o laço segue
  ([[Laço que trata toda falha igual apaga a causa da primeira]]).
- **Cobrar não é reenviar.** Reenvio repete o disparo (serve inclusive para o que
  falhou); cobrança pressupõe que o primeiro chegou. Misturar os dois foi o que deixou
  a RH sem saber se uma avaliação parada já tinha sido lembrada uma vez ou cinco.

## Conexões
- Princípio: [[Ausência só aparece contra o universo, nunca contra a tabela de eventos]]
- Irmã: [[Uma resposta canônica de um grupo é um token compartilhado]] ·
  [[Formulário público por token opaco fica fora do gate de sessão]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Backend]]
