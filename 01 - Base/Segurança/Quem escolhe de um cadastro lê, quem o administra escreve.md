---
tags: [tipo/atomica, camada/principio, seguranca, dev/backend]
criado: 2026-09-25
---

# Quem escolhe de um cadastro lê, quem o administra escreve

> Permissão por tela costuma dar o cadastro inteiro a uma tela só: Gestores é
> dona dos gestores, Formulários é dona dos formulários. Aí nasce a tela que só
> precisa ESCOLHER um gestor ou um formulário, e quem tem só ela recebe 403 na
> lista. As duas saídas óbvias estão erradas.

## A regra

**Ler um cadastro é direito de quem escolhe dele; escrever continua com quem o
administra.** A permissão do cadastro tem dois verbos, e cada um tem o seu dono.

As saídas erradas:

- **Dar a seção dona inteira** a quem só escolhe: resolve a lista e entrega junto
  criar, editar e apagar. Quem precisava escolher um formulário passa a poder
  apagar formulário.
- **Liberar pelo gate do módulo** (qualquer seção do módulo lê tudo): resolve e
  abre demais, e some a pergunta "quem precisa disto?".

## Por que

No RH do NaveX (herdado do nexo2), a Nova Avaliação de Desempenho escolhe
formulário, colaborador e setor; o envio de formulário escolhe gestores; a ficha
aberta na Rotatividade escolhe setor. As quatro listas eram de outras seções.
Com cargo de uma seção só, a tela abria e o seletor dava erro, o tipo de defeito
que só aparece quando alguém monta um cargo mais estreito que "admin".

O conserto ficou no registro de permissões, ao lado das seções donas: uma
entrada de leitura por endpoint ("setores é lido também por Desempenho,
Formulários e Rotatividade"), que o gate aceita só em GET. O endpoint de escrita
não mudou de dono.

No mesmo registro havia o furo do outro lado: um endpoint sem entrada nenhuma
(as regras de envio automático) caía no gate do módulo, e qualquer seção do RH
criava e apagava regra. Endpoint sem dono declarado é endpoint de todo mundo.

## Na prática

- Ao escrever uma tela que tem um seletor, pergunte de quem é a lista. Se não
  for da tela, a leitura cruzada é declarada no mesmo lugar das donas, com o
  motivo.
- A leitura cruzada vale pelo método (GET), não pela intenção da tela: o gate não
  sabe o que o front vai fazer com a resposta.
- Todo endpoint tem dono declarado. O que cai no padrão (módulo inteiro) é o
  primeiro lugar a revisar quando se aperta permissão.
- Se a lista carrega dado sensível que só o dono deveria ver, o seletor pede uma
  rota própria, magra (id e nome), em vez de ler o cadastro inteiro.

## Conexões
- Depende de: [[Permissão se valida no servidor, não na interface]]
- Irmã: [[Entidade auxiliar se cria no ponto de uso, não em tela própria]]
  (lá a tela que escolhe também cria; aqui ela só lê, e o porquê de cada caso)
- Visto em: [[NaveX]] · [[Navetech Hub]]
- Mapa: [[Base]]
