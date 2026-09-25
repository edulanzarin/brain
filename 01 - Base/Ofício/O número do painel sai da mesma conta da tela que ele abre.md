---
tags: [tipo/atomica, camada/principio, dev, armadilha]
criado: 2026-09-25
---

# O número do painel sai da mesma conta da tela que ele abre

> Todo painel de pendências é um atalho: o número diz "tem 29 aqui" e o clique
> leva à tela onde estão os 29. Se o painel contar por uma consulta e a tela
> listar por outra, as duas vão divergir, e a pessoa que clica acha 12. A
> partir daí ela para de confiar nos dois.

## A regra

**O número do painel é a contagem do que a tela mostra, feita pela mesma
função.** Não uma consulta "mais leve" escrita ao lado, não um `count(*)` direto
na tabela que a tela lê com filtro, não um status gravado que a tela recalcula
na hora.

A consulta própria do painel parece economia (a tela é pesada, o painel só quer
um número), e é exatamente aí que a regra se separa: a tela aprendeu, com o
tempo, o que NÃO entra na lista, e a consulta do painel nunca soube.

## Por que

Porque a tela é onde as correções acontecem. Quem descobre que o desligado não
deve aparecer, que o status vazio é pendente, que o atraso se mede pela data,
descobre olhando a tela, e conserta a tela. O painel, que ninguém lê linha a
linha, fica com a regra da primeira versão.

No porte do Nexo para o NaveX o mesmo defeito apareceu em dois módulos:

| Painel | Tela | O que divergia |
|---|---|---|
| Experiências a decidir: `count(*)` em `rh_experiencia` com status diferente de respondido | Monta os marcos a partir dos contratos em experiência no Questor | Quem foi desligado no meio da experiência ficava "a decidir" para sempre no painel e não existia na tela |
| Experiências em atraso: o status gravado `atraso` | Atraso pela data do vencimento | O status só mudava quando o job mandava o lembrete atrasado; sem lembrete, a tela dizia atraso e o painel não |
| eSocial pendente no DP | A pendência da tela | O painel e a tela tratavam o status vazio de jeitos diferentes |

Nos dois casos o conserto foi o mesmo: o painel chama a montagem da tela e conta
o resultado. No RH isso fez o painel passar a ler o Questor (antes lia só o
banco do app), e por isso a experiência virou um bloco próprio no painel: se o
Questor falhar, o resto do painel continua.

## Na prática

- **O painel importa a função da tela, não copia a consulta.** Se a tela é cara
  demais para o painel, a saída é uma função de contagem ao lado da de listagem,
  com o mesmo filtro extraído para um lugar só, nunca uma segunda consulta
  escrita do zero.
- **Status gravado que a tela recalcula não serve de contador.** Se a tela deriva
  a situação (pela data, pelo cruzamento com outra fonte), o painel deriva igual.
- **Conferência barata:** abrir o painel, clicar no número e contar as linhas.
  Divergiu, uma das duas regras está velha, e quase sempre é a do painel.
- Vale para qualquer resumo que leva a uma lista: o selo "3 novas" do menu, o
  total do e-mail diário, o contador da aba.

## Conexões
- Irmã: [[Dado escrito por dois caminhos precisa de uma regra só, fora dos dois]]
  (lá são dois caminhos de escrita; aqui, dois de leitura)
- Irmã: [[Ausência só aparece contra o universo, nunca contra a tabela de eventos]]
  (a tabela de trilha guarda o que aconteceu, não quem ainda está no universo)
- Irmã: [[Falta de registro só prova algo dentro da janela em que a fonte era alimentada]]
- Aplica em: [[A home de um módulo é o resumo que carrega sozinho; automação não abre sozinha]]
- Visto em: [[NaveX]] · [[Navetech Hub]]
- Mapa: [[Base]]
