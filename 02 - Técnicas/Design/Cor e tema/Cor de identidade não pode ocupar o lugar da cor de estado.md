---
tags: [tipo/atomica, camada/padrao, design, armadilha]
criado: 2026-09-02
---

# Cor de identidade não pode ocupar o lugar da cor de estado

> Tingir o cabeçalho da tabela com a cor do módulo parecia identidade barata —
> até o módulo cujo tom é coral. Cabeçalho rosa com texto vermelho, numa tabela
> de dinheiro, se lê como alarme, e não havia nada errado ali.

## O problema

Numa plataforma de vários módulos, cada um ganha uma cor e a tentação é
espalhá-la: cabeçalho, faixa, cartão, coluna. Funciona enquanto as cores
escolhidas são neutras entre si. Basta uma cair perto do vermelho, do âmbar ou
do verde para o significado atropelar a identidade — e o leitor não tem como
saber que aquele vermelho quer dizer "Contábil" e não "erro".

O mesmo defeito aparece do outro lado, na régua de valores: um componente de
dinheiro que colore por SINAL (verde acima de zero, vermelho abaixo) está certo
em saldo, variação e movimento de caixa. Numa coluna de **divergência** ele
mente: positivo ali não é bom, é diferença para cima, e o verde faz a célula
afirmar "está certo" ao lado do próprio ícone de alerta. A tela discordando de
si mesma na mesma linha.

## A solução

Duas fronteiras, ditas em voz alta no código:

- **Numa tela de número, cor é reservada para ESTADO.** A identidade do módulo
  mora no topo da página (o chip do cabeçalho, a barra lateral), não em cima do
  dado. O que separa o cabeçalho do corpo é um filete neutro e o peso da fonte —
  rótulo de coluna preto e forte, que é o que a régua da tabela precisa ser.
- **Colorir por sinal é opção, não padrão.** O componente aceita a coloração,
  mas a doutrina fica escrita nele: só onde subir é bom e descer é ruim. Onde a
  cor tem de medir GRAVIDADE (desvio pequeno × grande), ela não pode medir
  direção.

## O que mais vale lembrar

- **A cor afirma.** Uma paleta aplicada por regra genérica afirma coisa errada
  na primeira coluna que não é do tipo que a regra supôs — e afirma sem erro,
  sem log e com aparência de acabamento.
- **Escolher a cor de um módulo é escolher onde ela não pode entrar.** Vale
  decidir isso junto com a paleta, não quando a primeira tabela ficar rosa.

## Conexões
- Princípio: [[Todo estado da tela tem visual]]
- Irmã: [[Acento da interface é um token separado da cor de dado]] ·
  [[Cor de identidade não se drena por estado de disponibilidade]] ·
  [[Validar paleta de gráficos antes de escolher cores]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Design]]
