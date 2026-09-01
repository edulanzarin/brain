---
tags: [tipo/atomica, camada/principio, dev, armadilha]
criado: 2026-09-01
---

# Recurso sem escrita parece pronto quando a semente preenche a leitura

> Uma funcionalidade tem duas metades: o que a mostra e o que a cria. A que
> mostra é a que se vê, é a que dá prazer construir, e é a que a semente de
> desenvolvimento consegue satisfazer sozinha. Enquanto o banco vier semeado,
> um recurso com metade da leitura pronta e nenhuma escrita é
> indistinguível de um recurso terminado.

## A regra

**A conferência de que um recurso existe é o caminho de escrita, nunca a tela
de leitura.** A pergunta que revela a metade que falta não é "isso aparece?", é
**"que ação do usuário cria esta linha?"** — e ela se responde apontando a
função, não a tela.

Uma tabela que só é lida é uma tabela que ninguém escreve.

## Por que

Porque a ilusão é ativa, não passiva. A semente não deixa a metade faltante em
branco: ela a preenche com dado plausível, e o vazio passa por cheio. No
Privello, quatro recursos chegaram ao fim de um corte inteiro assim:

| O que existia | O que faltava |
|---|---|
| Nota média, resumo, portão do passe, cartão de avaliação | Nada criava uma avaliação — os números eram da semente |
| Página de favoritas lendo a tabela | O coração do perfil não tinha ação; a lista era vazia por construção |
| Trilho, visor e prazo de 24 h do story | Não havia como enviar um |
| Modal de denúncia, com torrada de sucesso | Nenhuma linha era escrita; a torrada mentia |

Os quatro sobreviveram a demonstrações, a revisões e a um "fecha ponta a
ponta". A denúncia é o caso mais duro porque ali nem a semente era necessária:
a tela dizia "enviada" e ninguém tinha como perceber.

## Na prática

- **Faça o inventário na direção da escrita.** Para cada tabela do schema,
  aponte a função que insere nela. A que não tiver é a lista do que falta, e ela
  se levanta em minutos.
- **Semente com dado plausível engana quem a escreveu.** Se a semente enche
  avaliações, ela precisa ter vindo pela mesma função que a tela usaria — e se
  essa função não existe, a semente acabou de esconder isso.
- **Botão sem ação é pior que botão ausente.** O ausente é uma lacuna que se
  vê; o presente afirma que o recurso existe, e cobra o preço na primeira
  pessoa de verdade que clicar.
- Vale para o portão do mesmo jeito: uma trava vendida como benefício e nunca
  exercida do lado de quem paga é uma trava que não abre —
  [[Permissão se valida no servidor, não na interface]].

## Conexões
- Irmã: [[Contador que conta sucesso de promessa afirma que deu certo]] — lá a
  métrica mente sobre o que aconteceu; aqui a tela mente sobre o que existe.
- Irmã: [[Um invariante se garante na estrutura, não no processo]]
- Técnica que aplica: [[Peça de mentira que não se anuncia vira fundação de coisa real]]
- Visto em: [[Privello]]
- Mapa: [[Base]]
