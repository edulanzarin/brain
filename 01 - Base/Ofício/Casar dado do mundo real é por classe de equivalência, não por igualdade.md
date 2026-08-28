---
tags: [tipo/atomica, camada/principio]
criado: 2026-08-28
---

# Casar dado do mundo real é por classe de equivalência, não por igualdade

> Telefone, nome, documento e endereço chegam ao sistema em mais de uma grafia — todas
> corretas, todas do mesmo alguém. Comparar por igualdade literal responde "não achei"
> onde a resposta certa era "achei", e o custo não é um erro visível: é um registro
> duplicado ou uma pessoa mandada caçar à mão o que o sistema tinha.

## A regra

Antes de comparar dois dados que vieram do mundo, defina a **classe de equivalência**:
o conjunto de grafias que significam a mesma coisa. Compare classes, nunca strings.

Dois caminhos, e a escolha é por qual lado você controla:

- **Canonizar os dois lados** quando a normalização é determinística — nome sem acento e
  em caixa alta, CNPJ só com dígitos, e-mail em minúscula.
- **Gerar as variantes e buscar em todas** quando a forma canônica não existe — o celular
  brasileiro com e sem o nono dígito não tem versão "certa", tem duas.

E a classe se justifica pelo **mundo**, não pela conveniência de quem busca: o 9 do
celular existe porque a operadora mudou o plano de numeração; o acento cai porque o
teclado do caixa não tem. Alargar sem um fato desses por trás não aumenta a chance de
achar — aumenta a de achar a pessoa errada.

## Por que

O erro é silencioso dos dois lados. Estreito demais, o sistema **duplica**: no
[[navetalks]], a mensagem que chegava do WhatsApp sem o 9 não encontrava o contato salvo
com o 9, e cada cliente virava dois registros com a conversa partida ao meio. Largo
demais, o sistema **funde**: casar pagamento com funcionário só por primeiro nome junta
todos os "João" do quadro numa pessoa só.

Nenhum dos dois aparece como falha. O primeiro parece cliente novo; o segundo parece
resposta.

## Na prática

- Comece pela grafia que o mundo produz, não pela que o seu banco guarda: é o extrato, o
  webhook e o formulário que definem a classe.
- Toda normalização é uma perda de informação deliberada. Perca só o que não discrimina
  (acento, máscara, caixa) e guarde a grafia original — ela ainda vai ser exibida.
- **Quando a classe fica larga a ponto de caber mais de uma pessoa, o casamento deixa de
  ser identificação e vira indício**: aí ele precisa de um segundo sinal que prove
  (documento) ou de entregar a dúvida a quem decide, em vez de escolher em silêncio.

## Conexões
- Técnica que aplica: [[Casar telefone brasileiro tolerando o nono dígito]] ·
  [[Casar o favorecido do extrato com a folha - CPF prova, nome indicia]] ·
  [[Dígito verificador rejeita o documento errado na entrada]]
- Mapa: [[Base]]
