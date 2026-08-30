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
- [[Filtrar no cliente ou no servidor se decide pelo tamanho, não pelo gosto]] — duas listas do mesmo site pedem soluções opostas; o que decide é quanto dado atravessa o fio e como esse tanto CRESCE. Página que engorda porque o projeto deu certo é filtro do lado errado.
  bruto; markup de card comprime ~95%, e 1,14 MB viraram 68 KB. Escolher o número
  errado faz reescrever componente pra resolver problema que não existe.
- [[router.replace do Next falha no build de produção]] — funciona em `dev`, falha
  calado em `build`. Princípio: [[Verificar no build de produção, não só em dev]].
- [[Arte servida sem hash de build precisa de versão na URL]] — arquivo em `public/` não
  é invalidado por republicar: quem já visitou serve a cópia velha, e o defeito só existe
  pra quem já visitou — no computador de quem publicou está tudo certo.
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
- [[Segmento de URL que começa com @ não chega ao App Router]] — `/@duda` devolve
  404 antes de a página rodar, codificado também; o `@` é marca de rota paralela e a
  reserva vale no endereço. A arroba é da tela, o endereço vai sem ela — e aí `/duda`
  e `/rs` convivem na raiz, porque sigla de estado tem duas letras e @ tem três.
- [[React reseta o formulário ao fim de uma Server Action]]
- [[A segunda ação do formulário se marca no botão, não no estado]] — apagar e salvar
  no mesmo formulário: `name`/`value` no botão clicado. Limpar o campo no `onClick`
  manda o valor antigo, porque o envio lê o campo de agora e o `setState` só chega no
  render seguinte.
- [[Janela arrastável no React 19 se faz à mão, não com react-rnd]] — react-rnd/
  react-draggable usam `findDOMNode`, removido no React 19; arraste/resize à mão,
  comitando no soltar.

- [[Portão de conteúdo cobre a tela, não o HTML]] — confirmação de maioridade,
  aviso de cookie, aceite de termos: se o portão decidir o que o servidor manda,
  toda URL do domínio devolve a mesma tela e o buscador lê um site vazio. Ele é
  overlay por cima de página inteira.

## Estado e renderização

- [[A regra que a server action executa mora fora dela]] — action fica com sessão,
  revalidate e redirect; a regra vira função comum que um script roda contra o banco,
  sem navegador. Folha isolada por enquanto: falta o princípio que a cubra.
- [[Cache do React Query não é lugar de estado de interface]]
- [[Portal condicional dispensa o flag de montagem]]
- [[Foto sem storage vira thumbnail data URL gerado no cliente]] — avatar/foto
  antes do storage: reduz no canvas, salva data URL, troca por upload depois.
- [[Trocar o arquivo repede a senha e relê os dados]] — estado derivado do arquivo
  se recalcula ao trocar a fonte, não se herda. Princípio:
  [[Um invariante se garante na estrutura, não no processo]].
- [[Número de resumo sai do mesmo cálculo que a tela detalha]] — cabeçalho que
  afirma "o melhor é X" e a lista abaixo são a mesma pergunta em dois tamanhos.
  Suba a conta pro pai e distribua o array pronto; um `filter` no pai com condição
  que já existe no filho é a segunda definição nascendo.

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
- [[Flutuante dentro de modal precisa vencer no z-index e no Escape]] — select e tooltip
  dentro de modal quebram de duas formas caladas: a lista abre atrás do véu (portais
  irmãos, o 100 cobre o 50) e o Escape fecha o modal inteiro (captura chama na ordem de
  REGISTRO, e o modal registrou antes). Flutuante vai pra cima do modal, e quem está por
  baixo pergunta se há camada acima antes de responder à tecla.
- [[Sem servidor, a migração de dado local acontece na leitura]] — com o dado no
  `localStorage` de quem usa, não há rotina de virada: a única hora de alcançar aquele
  navegador é quando ele abre a página. A chave velha entra na função que lê, só de
  leitura, e SOMADA à nova — substituir some com o que a aba antiga em cache ainda salva.

## Arquitetura de canvas / motor

- [[Mundo imperativo e React se falam por eventos, não por referência]] — engine
  (dados) / game / UI em camadas; o canvas emite eventos, o React escuta e manda
  intenção por controles estáveis. Candidato a princípio na 2ª aparição.

## TypeScript

- [[NoInfer faz o genérico sair da lista, não do valor padrão]]
- [[Componente que serve dois donos recebe o catálogo, não o campo renomeado]] — o
  segundo módulo não renomeia o campo dele pra caber no gráfico do primeiro: entram o
  catálogo, o acessor e o rótulo por prop, com o dono original como default. Traz a
  armadilha de tipo: só `type` ganha índice implícito, `interface` não.

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

## Saída de dado (arquivo que o usuário abre)

- [[CSV que abre no Excel pt-BR usa ponto e vírgula, BOM e vírgula decimal]] — os três
  detalhes de locale; o pior é o decimal, que faz a coluna chegar como texto e não
  somar sem parecer defeito.
- [[Traduza o vocabulário do sistema, não o nome próprio]] — em ferramenta PT sobre
  sistema EN, conceito traduz e nome próprio não; e a tradução mexe no índice de busca.

- [[Efeito que roda duas vezes destrói o que ainda não terminou de nascer]] — recurso
  criado com `await` dentro de efeito só se publica depois do init; a limpeza da
  primeira passada do React 19 em dev chega com a criação no ar e destrói pela metade.

---

Voltar para [[Início]]
