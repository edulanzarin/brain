---
tags: [tipo/atomica, camada/padrao, design]
criado: 2026-08-30
---

# Último passo sem desfecho transforma a régua em beco

> Uma régua de passos promete um fim só por existir: "Passo 9 de 9" afirma que
> existe algo do outro lado do nono. Se o rodapé do último passo tem o voltar à
> esquerda, o contador no meio e nada à direita, a promessa não é cumprida — a
> pessoa percorre o caminho inteiro e não descobre se acabou.

## O sintoma

É fácil não ver, porque cada passo foi construído olhando para o seguinte. O
último não tem seguinte, então o lugar do "avançar" fica vazio por omissão, não
por decisão. Quem testa o fluxo já sabe que acabou e não sente falta; quem
percorre pela primeira vez fica olhando a tela procurando o botão.

## A regra

**O último passo carrega o desfecho no mesmo lugar em que os outros carregam o
avançar.** Mesmo canto, mesmo peso visual — é a continuação da régua, não um
elemento novo.

E o desfecho quase nunca é um só: ele depende de onde a coisa ficou. No cadastro
de anúncio do Privello são três, e a tela escolhe qual mostrar:

| Situação | Desfecho |
|---|---|
| Falta algo que impede | "Resolver o que falta", apontando para a primeira falta |
| Está pronto e esperando terceiro | "Concluir", que fecha o caminho e volta ao painel |
| Já está publicado | "Ver meu perfil no ar" |

"Concluir" é o caso que costuma faltar: quando o fim do caminho não é uma ação
da pessoa e sim uma espera (uma moderação, uma aprovação, um processamento), o
desfecho é declarar que a parte dela terminou. Sem esse botão, o fim vira
inferência.

## Vale lembrar

Se o desfecho vai no rodapé, o botão avulso que fazia o mesmo papel no meio do
conteúdo sai. Dois botões rosa na mesma tela fazem a pessoa escolher qual é o
avançar — que é exatamente a dúvida que a régua existia para acabar.

## Conexões
- Princípio: [[Todo estado da tela tem visual]] — o fim do caminho é um estado, e
  estado sem desenho é a mesma ausência de resposta do "silêncio depois de salvar".
- Irmã: [[Tela que abre vazia tem que ensinar, tela que abre cheia não]]
- Visto em: [[Privello]]
- Mapa: [[Design]]
