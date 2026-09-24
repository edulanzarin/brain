---
tags: [tipo/atomica, camada/padrao, dev/backend]
criado: 2026-09-24
---

# Bot que qualquer um pode pôr num grupo se vincula a ele por código

> O nome de um bot do Telegram é público. Se o sistema aceita como "o grupo do
> cliente" o primeiro grupo onde o bot vira administrador, qualquer pessoa que
> promova o bot no próprio grupo desvia os convites pagos para lá.

## O problema

O jeito óbvio de descobrir o grupo é ouvir o `my_chat_member`: o bot foi
promovido a administrador em algum chat, então aquele é o grupo. Funciona no
teste, porque no teste só o dono adiciona o bot. Em produção, o `@username` do
bot está no link de venda que o criador divulga, e o update chega igual para
qualquer grupo.

## A solução

Cada bot ganha um código curto (sem caracteres ambíguos), mostrado só no painel
logado. O vínculo só acontece quando o código volta pelo Telegram:

- **Grupo**: o botão do painel abre
  `https://t.me/<bot>?startgroup=<CODIGO>&admin=invite_users+restrict_members`.
  O Telegram oferece escolher o grupo, já sugere as permissões de administrador,
  e ao adicionar entrega ao bot a mensagem `/start@<bot> <CODIGO>` dentro do
  grupo. Código certo, grupo vinculado.
- **Canal** não aceita payload no link de adicionar. O criador publica no canal
  `/vincular <CODIGO>`, e o bot (administrador) recebe como `channel_post`.
- **Dono**: `https://t.me/<bot>?start=dono-<CODIGO>` numa conversa privada grava
  o id do Telegram do criador, para ele receber aviso de venda.

Depois do vínculo, `my_chat_member` do grupo vinculado só serve para reler as
permissões (`getChatMember` do próprio bot): promover o bot em outro grupo não
muda nada.

## O que mais vale lembrar

- As permissões se releem da fonte (`getChatMember`) de tempos em tempos, e não
  só no vínculo: alguém rebaixa o bot por fora e o painel precisa saber antes do
  primeiro comprador pagar e não receber convite.
- Com o vínculo feito por código, o checklist de ativação consegue dizer
  exatamente o que falta ("o bot está em X, mas não pode banir usuários").
- O código aparece também no painel como texto copiável, porque o caso do canal
  exige digitar.

## Conexões
- Princípio: [[Afirmação que chega de fora só vale com um código que a casa emitiu antes]]
- Irmã: [[Código que a pessoa copia à mão não pode ter caractere ambíguo]] ·
  [[Estado mutável se lê da fonte no uso, não de cópia guardada]]
- Visto em: [[telebot]]
- Mapa: [[Backend]]
