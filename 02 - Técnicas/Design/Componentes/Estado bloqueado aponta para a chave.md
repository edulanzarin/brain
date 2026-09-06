---
tags: [tipo/atomica, camada/padrao, design, armadilha]
criado: 2026-09-01
---

# Estado bloqueado aponta para a chave

> "Contato só para assinante", "disponível no plano Pro", "faça login para
> continuar": o estado bloqueado é fácil de desenhar e fácil de deixar pela
> metade. Ele informa que existe um cadeado e não diz onde fica a chave — e
> para quem está olhando, um controle que anuncia algo e não leva a lugar
> nenhum é indistinguível de um controle quebrado.

## O problema

O bloqueio nasce como negação, e negar é uma linha:

```tsx
if (fechado) {
  return <span className="botao">Contato só para assinante</span>;
}
```

Um `<span>` no lugar de um botão. Ele desenha certo, alinha com os vizinhos e
não faz nada. Quem chegou ali é justamente a pessoa mais disposta a pagar
naquele segundo — e o que ela recebe é o fim da conversa.

O erro tem duas variações piores:

- **A chave que aponta para o lugar errado.** No Privello, o vazio de avaliações
  oferecia "Conhecer o passe" e levava para `/anunciar`, a página de quem
  anuncia. Quem procurava caía numa tela sobre planos de anúncio e concluía que
  tinha entendido errado.
- **O cadeado decorativo.** O selo "Exclusivo" ficava por cima da foto
  verdadeira, enviada ao navegador como qualquer outra. Não era portão, era
  adesivo — e o que ele bloqueava de verdade era só a boa consciência de quem o
  desenhou. Portão é do servidor:
  [[Permissão se valida no servidor, não na interface]].
- **O botão desligado.** Pior que o `<span>`, porque o `disabled` é uma
  afirmação: diz que o controle existe e que agora não serve. Não recebe foco,
  não recebe toque, e leitor de tela anuncia "indisponível" — a pessoa nem
  descobre que havia o que comprar.

## A forma

O estado bloqueado é **um link**, com o mesmo peso do controle que ele
substitui, e o destino é onde se compra o que abre aquilo:

```tsx
if (fechado) {
  return (
    <Link href={ondeAssinar} className="botao">
      <Icone nome="cadeado" />
      Contato só para assinante
    </Link>
  );
}
```

O destino chega por propriedade e não fica cravado na peça: quem sabe onde a
chave se vende é a tela, não o botão.

## Consertar num lugar não conserta nos outros

No Privello o botão do perfil já era um link para o passe, com o motivo escrito
no comentário. O cartão da vitrine, escrito depois, repetiu o erro inteiro em
outra forma — `disabled` com "Só assinante" —, e ficou meses assim ao lado da
versão certa.

O que faz a regra viajar não é o comentário no componente vizinho: é a peça
aparecer no catálogo com o estado bloqueado montado, onde quem vai escrever o
próximo tropeça nela antes de inventar o seu.
[[Catálogo de componentes é contrato vivo, não documentação]].

## O que mais vale lembrar

- **Três estados, não dois.** Sem conta, com conta e sem o benefício, e com o
  benefício. Os dois primeiros levam a lugares diferentes, e tratá-los como um
  só manda quem não tem conta para a tela de compra e quem já tem para o login.
- **Bloqueio no próprio conteúdo também.** Se o benefício é ler algo, quem não
  tem vê o espaço reservado com a chamada dentro dele, não um buraco.
- **O bloqueio que a pessoa causou em si mesma se avisa antes.** Quem anuncia e
  liga "atender só assinante" precisa saber, no mesmo cartão, se existe passe à
  venda — senão a chave que ela trancou não tem par no mundo.

## Conexões
- Princípio: [[Todo estado da tela tem visual]] — bloqueado é um estado, e estado
  desenhado pela metade é o mesmo silêncio de estado nenhum.
- Irmã: [[Último passo sem desfecho transforma a régua em beco]] — o beco do fim
  do caminho; este é o beco do meio.
- Irmã: [[Permissão se valida no servidor, não na interface]]
- Visto em: [[Privello]]
- Mapa: [[Design]]
