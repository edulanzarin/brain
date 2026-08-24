---
tags: [tipo/moc]
criado: 2026-07-20
---

# Frontend

Técnicas e armadilhas presas ao React e ao Next. É a camada mais volátil do cérebro —
quando a ferramenta muda, estas notas envelhecem, e tudo bem. O que precisa sobreviver
mora em [[Base]].

Stack atual: Next.js (App Router) · React · TypeScript · Tailwind.

## Armadilhas do React e do Next

- [[Peso de página se mede no fio, não na saída do render]] — `wc -c` devolve o
  bruto; markup de card comprime ~95%, e 1,14 MB viraram 68 KB. Escolher o número
  errado faz reescrever componente pra resolver problema que não existe.
- [[router.replace do Next falha no build de produção]] — funciona em `dev`, falha
  calado em `build`. Princípio: [[Verificar no build de produção, não só em dev]].
- [[Componente de ícone não atravessa a fronteira server-client]]
- [[Componente de terceiro que usa Context não roda em Server Component]] — ícone/lib
  com `useContext` quebra no server; usar a entrada `/ssr` context-free.
- [[Slot de anúncio no App Router precisa de casca estável e filho keyado]] — o nó do
  anúncio só é reivindicado uma vez; entre rotas de mesma forma o React reconcilia e
  ele sobrevive com o anúncio velho dentro. Casca estável + filho keyado por rota.
- [[Token de cor que não existe vira cor herdada, sem erro]] — ao portar módulo entre
  projetos os tokens são fronteira: `var(--green)` sem o token cai pro valor herdado,
  fica legível e errado, e nada no console avisa.
- [[Altura 100% em item de grid de linha automática volta ao tamanho intrínseco]] — a
  linha depende do item e o item da linha; o navegador desiste do 100% e o conteúdo
  vaza da caixa de tamanho fixo. Só aparece com conteúdo não-quadrado.
- [[React reseta o formulário ao fim de uma Server Action]]
- [[Janela arrastável no React 19 se faz à mão, não com react-rnd]] — react-rnd/
  react-draggable usam `findDOMNode`, removido no React 19; arraste/resize à mão,
  comitando no soltar.

## Estado e renderização

- [[Cache do React Query não é lugar de estado de interface]]
- [[Portal condicional dispensa o flag de montagem]]
- [[Foto sem storage vira thumbnail data URL gerado no cliente]] — avatar/foto
  antes do storage: reduz no canvas, salva data URL, troca por upload depois.
- [[Trocar o arquivo repede a senha e relê os dados]] — estado derivado do arquivo
  se recalcula ao trocar a fonte, não se herda. Princípio:
  [[Um invariante se garante na estrutura, não no processo]].

Princípios: [[Estado compartilhável mora na URL]] ·
[[Estado de tela pertence à seção, não à página]]

## Fronteira do navegador (origem, storage)

- [[Sessão de outro domínio só se injeta rodando na origem dele]] — storage é isolado por
  origem; pra logar noutro domínio, o código roda lá (console/bookmarklet), não na sua
  página. REST Bearer server-side não tem essa trava. Candidato a princípio na 2ª aparição.
- [[Trocar de sujeito na mesma rota não remonta, e o estado do anterior fica]] — mudar só
  a query (`?conta=X`) mantém o componente montado: todo `useState` sobrevive e todo
  `useEffect(…, [])` não roda de novo. O próximo "salvar" grava os valores do sujeito
  anterior no novo, sem erro nenhum. `key` no sujeito resolve todos os filhos de uma vez.
  Princípio: [[Estado de tela pertence à seção, não à página]].
- [[O empacotador segue o valor importado, não o tipo]] — componente de cliente pode
  importar `type` de módulo de servidor à vontade; importar um VALOR do mesmo arquivo
  arrasta o módulo e as dependências dele pro navegador. O erro cita `net` e `fs`, nunca
  o seu arquivo. O contrato entre motor e tela vira módulo próprio.

## Arquitetura de canvas / motor

- [[Mundo imperativo e React se falam por eventos, não por referência]] — engine
  (dados) / game / UI em camadas; o canvas emite eventos, o React escuta e manda
  intenção por controles estáveis. Candidato a princípio na 2ª aparição.

## TypeScript

- [[NoInfer faz o genérico sair da lista, não do valor padrão]]

## Assets e geração

- [[Personagem pixel direcional se desenha em código, não se gera por IA]] — 4
  direções + walk coerentes: desenhe em código (corpo por direção + pernas por
  fase), sem depender de API. Princípio: [[Coerência em geração vem de âncora, não de liberdade]].
- [[PixelLab só mantém o personagem ao animar com image guidance alto]] — se for de
  IA mesmo: animar a partir de referência deriva no default; peso alto prende a
  identidade.

## Fronteira com o Design

O que é **aparência** (token, componente, estado visual) mora em [[Design]]. O que é
**comportamento do framework** mora aqui. Na dúvida: se continua verdade trocando o
Tailwind por outro CSS, é Design; se depende do React, é Frontend.

---

Voltar para [[Início]]
- [[Traduza o vocabulário do sistema, não o nome próprio]] — em ferramenta PT sobre
  sistema EN, conceito traduz e nome próprio não; e a tradução mexe no índice de busca.
