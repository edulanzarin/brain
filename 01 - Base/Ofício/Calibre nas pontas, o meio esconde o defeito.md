---
tags: [tipo/atomica, camada/principio, armadilha]
criado: 2026-08-25
---

# Calibre nas pontas, o meio esconde o defeito

> Ao ajustar um sistema que responde a um parâmetro, calibre no **menor e no
> maior valor** que esse parâmetro pode ter. O caso médio quase sempre fecha, e
> é justamente por fechar que ele engana: o defeito mora nas pontas, e a ponta
> de baixo costuma ser exatamente onde o usuário novo entra.

## O problema

Calibrar no caso típico é o reflexo natural: você monta o cenário representativo,
mexe nas constantes até o número ficar bom, e declara o sistema equilibrado. O
cenário representativo tem um vício: ele é representativo do **usuário que já
está dentro**.

O erro é sutil porque nada indica falha. Os testes passam, a tabela de resultados
fica plausível, o gráfico sobe. Só que a fórmula que você equilibrou tem um termo
que depende do parâmetro, e no extremo esse termo domina tudo.

## Por que a ponta de baixo é a pior

Nas pontas os efeitos se **compõem** em vez de se cancelar. Quem tem menos de um
recurso normalmente tem menos dos outros também, porque os recursos crescem
juntos. Então o pior caso não é uma variável no mínimo: são todas ao mesmo tempo,
e o resultado não é ruim proporcionalmente — é ruim ao quadrado.

E a ponta de baixo é onde alguém encosta primeiro. Quem chega agora tem o mínimo
de tudo, por definição.

## Como aplicar

1. Liste os parâmetros que o sistema aceita — tamanho, nível, quantidade, idade
   da conta, volume de dado.
2. Para cada um, rode o extremo inferior e o superior, não a média.
3. Faça isso **antes** de fechar as constantes, não depois de reclamarem.
4. Trave a ponta com teste. O teste do caso médio não pega regressão de ponta:
   ele continua verde enquanto a borda apodrece.

## Conexões
- Irmã: [[Um invariante se garante na estrutura, não no processo]]
- Visto em: [[Vespéria]]
- Mapa: [[Base]]
