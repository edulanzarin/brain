---
tags: [tipo/atomica, camada/principio, armadilha]
criado: 2026-09-25
---

# Especificidade se mede pelo que o campo distingue, não pelo tamanho do termo

> Quando várias regras casam com o mesmo registro, "o termo mais longo ganha"
> só funciona dentro de um campo. Com dois campos de naturezas diferentes, um
> que diz o TIPO e se repete em tudo, outro que diz QUEM e muda a cada linha,
> o termo curto no segundo é mais específico que o longo no primeiro. O tamanho
> mede o texto; quem mede a especificidade é quantos registros o campo separa.

## O caso

Extrato bancário: o histórico é abreviado e igual para toda transferência
("DÉB.TRANSF.CONTAS DIF.TITULARIDADE"), e o favorecido vem numa segunda linha
("FAV.: FULANO"). Com as regras lendo as duas linhas, "DEB.TRANSF.CONTAS
DIF.TITULARIDADE" (34 letras) ganhava de "VANIO" (5) pelo critério do tamanho,
e toda distribuição de lucro ia para a conta de um sócio só. A regra que só
casa porque leu o favorecido tem que subir um degrau acima de qualquer termo
do histórico. O desempate pelo tamanho continua valendo, só que dentro de cada
degrau.

## As outras duas decisões que vêm junto

- **Campo novo no casamento não pode apagar regra velha.** Regra "exata" feita
  quando só existia o histórico continua valendo contra o histórico sozinho; a
  linha inteira é uma segunda chance de casar, não a única. Senão ligar o campo
  novo desfaz, calado, o cadastro de meses.
- **O termo sugerido erra para o lado que aparece.** Criar a regra a partir de
  uma linha pré-preenche o termo. Pelo campo genérico, o erro é largo demais:
  a regra engole as linhas dos outros e as manda para a conta errada, sem
  pendência nenhuma. Pelo campo que distingue, o erro é estreito demais: a
  linha do mês que vem fica pendente e alguém olha. Entre um erro calado e um
  que aparece, o padrão fica com o que aparece.

## Onde mais vale

Qualquer classificação por regra sobre registro com campo de tipo e campo de
identidade: e-mail (assunto contra remetente), lançamento contábil (histórico
padrão contra complemento), chamado de suporte (categoria contra cliente). O
teste é perguntar quantos registros o campo separa, não quanto texto o termo
tem.

## Conexões
- Irmã: [[Casar dado do mundo real é por classe de equivalência, não por igualdade]]
  (estreito demais duplica, largo demais funde: a mesma tensão, do lado da identidade)
- Irmã: [[Ausência de leitura cai no valor que dispara a ação]]
  (o padrão errado não cai num ponto qualquer: cai no que age calado)
- Técnica: [[Ler extrato bancário em PDF]] (de onde vem o complemento)
- Visto em: [[NaveX]] (Conciliação bancária, set/2026)
- Mapa: [[Base]]
