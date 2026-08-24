---
tags: [tipo/atomica, camada/principio, dev/backend]
criado: 2026-08-24
---

# Produtividade se mede pela hora do registro, não pela data do fato

> Todo dado de trabalho carrega duas datas — quando o fato aconteceu e quando alguém o registrou. Medir gente é somar pela SEGUNDA; medir o negócio é somar pela primeira.

## A regra

Uma linha de sistema transacional quase sempre tem o par: `data do fato` (competência, data da nota, data de admissão, data do lançamento contábil) e `data/hora do registro` (o carimbo de auditoria de quando a linha nasceu). São perguntas diferentes:

- **Quanto o negócio movimentou em maio?** → data do fato.
- **Quanto a equipe trabalhou em agosto?** → hora do registro.

Trocar as duas parece detalhe e não é: lançamento de maio digitado em agosto é trabalho de agosto. Somado pela data do fato, ele aparece no mês que já foi entregue — e some do mês em que a pessoa efetivamente sentou e fez.

## Por que

O erro é silencioso: o número sai bonito, ninguém percebe. Quem fecha um mês atrasado (o normal em escritório contábil, onde julho fecha em agosto) some da própria produtividade; e o mês antigo "ganha" trabalho depois de já ter sido avaliado. Pior: o placar fica **instável**, porque a mesma consulta rodada duas vezes dá números diferentes para o mesmo mês passado.

O sinal de que a escolha está errada é esse: **produtividade de período fechado não pode mudar**. Se muda, você está somando pela data do fato.

## Na prática

- Verifique se a fonte tem o carimbo antes de prometer a tela: sem `datahora*`, não há placar de pessoa — só de negócio.
- O carimbo costuma ter índice próprio (é coluna de auditoria consultada por período), então filtrar por ele não é mais caro.
- Deixe explícito na interface qual data recorta o período. Quem olha assume competência por hábito.
- O mesmo par aparece fora da contabilidade: chamado (aberto × resolvido), pedido (emitido × faturado), tarefa (vencimento × conclusão).

## Conexões
- Irmã: [[Auditar o registro, não só o agregado]]
- Irmã: [[Rendimento é vazão vezes tempo em pé, não vazão de pico]]
- Técnica que aplica: [[Grão fino numa varredura só dispensa os count distinct]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Base]]
