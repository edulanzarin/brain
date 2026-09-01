---
tags: [tipo/atomica, camada/principio, dev, armadilha]
criado: 2026-09-01
---

# Dado escrito por dois caminhos precisa de uma regra só, fora dos dois

> Quase todo campo que importa acaba tendo dois caminhos de escrita: o cadastro
> que o cria e a tela que o corrige depois. Enquanto a regra dele estiver
> escrita dentro de cada um, os dois vão divergir — e a divergência é silenciosa,
> porque cada lado continua funcionando sozinho.

## A regra

**Uma leitura por campo, chamada pelos dois caminhos.** O cadastro pergunta
tudo de uma vez e a edição pergunta um bloco por vez, mas o que vale num preço
não pode depender de por qual tela ele entrou.

A função que valida não escreve e não conhece a requisição: recebe o que veio,
devolve o valor pronto ou a recusa com o nome do campo. Escrever fica de fora,
porque escrever é o que muda entre criar e corrigir.

## Por que

Porque a divergência não dá erro. Os dois caminhos passam nos próprios testes,
e o que muda é qual dos dois aceita o que o outro recusa. **O lado que aceita é
sempre o escrito depois**, e ninguém compara os dois de novo.

O sinal de alerta é a segunda tela: no dia em que se escreve o editor, a regra
do cadastro é copiada para lá. Naquele minuto as duas são idênticas e a cópia
parece inofensiva — e é exatamente ali que ela devia ter virado uma função.

No Privello, a mesma falha apareceu por quatro portas na mesma sessão:

| O que estava em dois lugares | O que teria divergido |
|---|---|
| A validação de preço, WhatsApp e descrição | O cadastro recusaria o que o painel aceitava |
| A conta de idade a partir do nascimento | A virada do aniversário caindo em dias diferentes: o formulário recusando uma idade que a página já exibia |
| A montagem do endereço do perfil | Uma das cópias invalidava `/uf/cidade/@arroba`, um caminho que o roteador nem serve |
| A contagem regressiva de 24 h | Recado e story arredondando diferente para a mesma hora |

O terceiro é o mais instrutivo: a cópia errada não quebrou nada visível. Ela
simplesmente deixou de fazer efeito, calada, por meses.

## Na prática

- **A duplicação nasce na hora da segunda tela, não na primeira.** Ao escrever
  o editor de algo que o cadastro já criava, o primeiro movimento é extrair, não
  copiar.
- **Formato conta como regra.** Se um lado guarda só dígitos e o outro guarda
  com máscara, os dois estão certos e o dado ficou inconsistente.
- **Derivar também.** Idade a partir de nascimento, prazo a partir de criação,
  endereço a partir de campos: são regras, e valem a mesma extração.
- O que sobra em cada caminho é o que é mesmo dele: quem pode escrever, o que
  invalidar, para onde ir depois — ver [[A regra mora fora da porta que a chama]].
- **A conferência ganha de graça.** Com a leitura fora das duas telas, um script
  exercita as duas de uma vez, sem navegador.

## Conexões
- Depende de: [[A regra mora fora da porta que a chama]] — lá o corte é entre
  regra e requisição; aqui é entre a regra e os vários caminhos que a chamam.
- Irmã: [[Estado mutável se lê da fonte no uso, não de cópia guardada]] — a mesma
  doença um nível abaixo: lá a cópia é do dado, aqui é da regra.
- Irmã: [[A definição em dado dirige o comportamento, não um caso no código]]
- Visto em: [[Privello]]
- Mapa: [[Base]]
