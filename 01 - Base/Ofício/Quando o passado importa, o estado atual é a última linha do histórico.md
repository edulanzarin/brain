---
tags: [tipo/atomica, camada/principio]
criado: 2026-09-25
---

# Quando o passado importa, o estado atual é a última linha do histórico

> Se alguém vai perguntar "com quem estava em março?", o "com quem está agora"
> não é uma coluna do cadastro: é a última linha de um histórico em que nada se
> reescreve. Guardar os dois é abrir espaço para o cadastro dizer uma coisa e o
> histórico outra, e o histórico é o que precisa ser defendido.

## O que decorre disso

- **O "de onde veio" não se grava.** Cada linha diz só para onde foi; de onde
  veio é o destino da linha anterior. Gravar os dois lados em cada linha é a
  mesma duplicação em escala menor, e uma correção num lado desmente o outro.
- **Linha nova não entra no meio.** Uma movimentação datada antes da última
  reescreveria o "de onde veio" da seguinte. Retroativo vale, mas só a partir
  do dia da última linha; antes disso, o histórico já tem dono.
- **Duas escritas ao mesmo tempo não podem criar duas "últimas".** A
  transação trava a entidade antes de ler onde ela está e de gravar a linha
  nova.
- **O que a linha cita de fora fica congelado nela.** O nome e o setor de quem
  recebeu vão para a linha na hora, porque a fonte esquece: quem sai da
  empresa some do cadastro de pessoas, e o histórico continua precisando dizer
  com quem o item ficou. É o mesmo movimento de
  [[O acordo congela na linha, a política vale do próximo em diante]].
- **Apagar e encerrar são coisas diferentes.** Enquanto só existe a linha do
  cadastro, apagar corrige um engano. Depois que o item passou por alguém, o
  fim dele é uma linha a mais (baixa), que guarda o caminho até ali.
- **Em lote, é tudo ou nada.** Entregar um kit ou devolver tudo no desligamento
  é um gesto só; metade gravada deixa o histórico descrevendo algo que ninguém
  fez. A recusa diz quais itens impediram.

## O custo

Ler o estado passa a ser "a última linha de cada entidade", que pede um índice
por (entidade, data, id) e uma junção lateral. Em centenas ou milhares de itens
é imperceptível. Se um dia pesar, a coluna de estado pode existir como cópia,
escrita só na mesma transação que grava a linha e em nenhum outro lugar; a
fonte continua sendo o histórico.

## Conexões
- Irmã: [[Dado escrito por dois caminhos precisa de uma regra só, fora dos dois]]
- Irmã: [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[O acordo congela na linha, a política vale do próximo em diante]]
- Visto em: [[NaveX]] (TI, equipamentos e com quem está cada um, set/2026)
- Mapa: [[Base]]
