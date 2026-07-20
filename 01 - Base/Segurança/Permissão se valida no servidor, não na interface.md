---
tags: [tipo/atomica, camada/principio, seguranca, dev/backend]
criado: 2026-07-20
---

# Permissão se valida no servidor, não na interface

> Esconder o botão é conveniência visual. A regra de acesso mora no endpoint, e o
> dado que o usuário não pode ver nunca chega ao navegador dele.

## A regra

Duas checagens, com papéis diferentes e não intercambiáveis:

- **No cliente**, esconder o que não se aplica — pra não oferecer ação que vai falhar.
  Isso é usabilidade.
- **No servidor**, negar — em toda rota, sempre, mesmo nas que a interface "garante"
  que só o admin alcança. Isso é a segurança.

E o principal: **dado sensível não vai junto na resposta pra ser escondido no front**.
No [[Cofre Digital]] a senha de um acesso nem chega ao navegador dos setores de leitura;
só o endpoint dedicado de cópia a entrega, depois de checar o perfil. Campo oculto por
CSS é campo visível no DevTools.

## Por que

A interface é território do usuário — ele tem o código-fonte, o DevTools e a rede. Toda
regra que só existe no front é uma sugestão. O front esconde a porta, mas quem tranca é
o servidor.

## Perfil por módulo, não por cargo

Amarrar permissão ao cargo ("setor financeiro pode X") parece natural e endurece rápido:
a primeira exceção legítima obriga a criar um cargo falso. Melhor separar as duas
coisas: **setor é rótulo** (quem a pessoa é), **perfil é regra** (`view`/`edit` por
módulo). Assim uma exceção é um perfil ajustado, não uma pessoa no cargo errado.

## Conexões
- Padrão que aplica: [[Servir anexo por rota com checagem de permissão]]
- Irmã: [[Configuração vem do ambiente, não do código]]
- Mapa: [[Base]]
