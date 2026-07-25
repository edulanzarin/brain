---
tags: [tipo/pensamento, design]
criado: 2026-07-25
---

# Interface de cliente fala pouco e esconde o backend

> Sistema de cliente é elegante e quieto: não narra a própria funcionalidade, não
> mostra jargão de backend na tela, e corta todo texto que o usuário já deduz. Um
> monte de texto explicando o que o sistema faz é paia.

## O pensamento

O Eduardo quer que a interface de um produto de cliente **diga o mínimo**. Duas
coisas o incomodam na tela:

- **Texto que explica funcionalidade.** Subtítulo de modal que repete o título,
  toast que narra o efeito colateral ("Empresa criada, o cofre dela já está
  disponível"), descrição que reafirma o óbvio. Rótulo forte dispensa explicação;
  o essencial se infere. Ele cortou dezenas dessas de uma vez e o sistema ficou
  mais elegante, não menos claro.
- **Jargão de backend vazando pra tela.** `.env`, "banco de dados", "container",
  "Docker", uid, "migração" numa tela que o cliente usa. Esse vocabulário é do
  time que instala, não de quem opera. Vai pro código e pra doc de infra, nunca
  pro rosto do usuário. (Ex.: o campo de pasta de arquivos falava de `.env` e
  container; virou "Senha de segurança" e "Mover os arquivos existentes".)

Detalhe de estilo que entra junto: nada de travessão espalhado por frase. Frase
curta, ponto. É a mesma voz enxuta de [[Detalhe em modal, linha enxuta]], só que
no texto em vez de no layout.

## Por que importa

Interface que se explica o tempo todo trata o usuário como se ele fosse burro e
ainda suja a tela. Menos texto varre mais rápido, parece mais acabado e passa
confiança. E jargão técnico exposto não é transparência: é vazamento de
implementação, deixa o produto com cara de ferramenta interna meio pronta.

Outra pessoa competente pode preferir uma UI mais "tutorial", com mais dicas na
tela — por isso é pensamento, não princípio. Mas para os produtos dele a escolha
é a voz quieta.

## Conexões
- Irmã: [[Detalhe em modal, linha enxuta]]
- Visto em: [[Cofre Digital]]
- Mapa: [[Pensamentos]]
