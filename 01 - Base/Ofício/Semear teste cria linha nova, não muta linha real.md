---
tags: [tipo/atomica, camada/principio, dev, armadilha]
criado: 2026-07-20
---

# Semear teste cria linha nova, não muta linha real

> Para verificar algo contra um banco com dados reais, semeie linhas descartáveis.
> Nunca altere uma linha que já existe: a limpeza apaga o que você criou, mas não
> desfaz o que você mudou.

## O quê

Verificar uma feature muitas vezes precisa de dado que ainda não existe (um grupo,
um vínculo, um registro de exemplo). A tentação é reaproveitar uma linha real que já
está lá — pegar as duas primeiras empresas e pô-las num grupo de teste, por exemplo.

O problema aparece na hora de limpar. "Apaguei o grupo de teste" desfaz a *criação*,
mas o vínculo que eu sobrescrevi não volta: com `onDelete: SetNull`, apagar o grupo
zera o `groupId` da empresa — e ela não tinha `groupId` nulo antes, tinha o grupo de
verdade do dono. A limpeza não restaura o valor anterior porque esse valor foi
perdido no momento em que sobrescrevi.

Regra: **semear é INSERT de linha descartável, nunca UPDATE de linha real.** Se não dá
para evitar o UPDATE, leia o valor original antes e restaure-o no fim — a limpeza
passa a ser simétrica à mutação, não só ao insert.

## Por que importa

O modo silencioso é o pior: o teste passa, a limpeza "roda", e o dado do usuário
sumiu sem erro nenhum. Só se descobre quando ele nota que a vinculação que tinha feito
evaporou. Contra banco de produção/real, o dano de sobrescrever é irreversível de um
jeito que criar-e-apagar nunca é.

Caso concreto: verificando a coluna de grupo do [[Cofre Digital]], pus empresas reais
num grupo de teste e depois apaguei o grupo; o `SetNull` deixou as empresas sem grupo,
apagando a vinculação "2RXD → Teste" que o Eduardo tinha feito à mão.

## Conexões
- Irmã: [[Verificar no build de produção, não só em dev]]
- Visto em: [[Cofre Digital]]
- Mapa: [[Base]]
