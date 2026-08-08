---
tags: [tipo/moc]
criado: 2026-07-20
---

# Design

**O que se reusa entre projetos é o princípio, não a estética.** Os princípios (por
quê) moram em [[Base]] e sobrevivem a qualquer redesign. As técnicas daqui (como) são
um *dialeto* concreto — o dashboard denso "Aurora Glass" — traduzido em CSS, Tailwind
e React, ramificado por assunto. É uma opção de cara, boa pra sistema de trabalho tipo
dashboard, não uma linguagem que todo projeto tenha que herdar:
[[Estética é por projeto, princípio de design é que se reusa]].

Projeto novo herda os princípios sempre; puxa este dialeto só se for da mesma forma
(admin/dashboard). Landing, app de cliente ou mobile escolhem a própria estética.

## Cor e tema

`02 - Técnicas/Design/Cor e tema`

- [[Sistema de cores e tema do dashboard]] — os tokens semânticos, claro/escuro e o
  script que evita o flash no carregamento.
- [[Vidro flutuante precisa de superfície mais opaca que a chrome]] — dois níveis de
  vidro: chrome arejada vs overlay opaco e legível.
- [[Validar paleta de gráficos antes de escolher cores]] — forma primeiro, cor por
  último; checagens objetivas antes de fechar a paleta.
- [[Cor de marca precisa de variante acessível por tema]] — a cor crua do cliente
  reprova contraste como texto; derive uma variante por tema e compute.

Princípios: [[Token semântico em vez de valor literal]] ·
[[Hierarquia por superfície, não por borda]]

## Layout e espaço

`02 - Técnicas/Design/Layout e espaço`

- [[Classes de componente vão em @layer components no Tailwind]] — pra a classe de
  componente vencer a utilitária sem `!important`.

Princípios: [[Escala fechada em vez de valor solto]] ·
[[Container tem largura máxima e respiro constante]]

## Componentes

`02 - Técnicas/Design/Componentes` — índice: [[Padrões de componentes de dashboard]]

- [[Sidebar em acordeão e layout de módulo]] — a estrutura fixa da tela.
- [[Controles de filtro do dashboard]] — toggle segmentado e dropdown de filtro.
- [[Seletor cria e gerencia os próprios itens]] — combobox que cria ao digitar e
  renomeia/exclui no painel, dispensando tela de CRUD do auxiliar.
- [[Blocos de dado - card, KPI e gráfico]] — card, stat tile, gráfico e tabela.
- [[Modal com conteúdo que cresce tem teto de altura e área que rola]] — overlay não
  pode crescer sem fim: teto no painel, scroll na parte que cresce.
- [[Primitiva de botão fecha o tamanho e abre só a variante]] — o botão vira
  componente que expõe cor, não tamanho; o `!h-7` por instância deixa de existir.
- [[Barra de topo contextual - o módulo injeta suas ferramentas via portal]] — o
  topo muda por tela: cada módulo manda busca/ações via slot, sem busca duplicada.

Princípios: [[A variante de um controle muda a intenção, não o tamanho]]

## Estados e feedback

`02 - Técnicas/Design/Estados e feedback`

- [[Toast em vez de alert para o feedback do app]] — canal de mensagem próprio.
- [[Esqueleto de carregamento imita a forma do conteúdo]] — carregar sem saltar.
- [[Filtro de lista mora na URL]] — estado que sobrevive ao reload.
- [[Consulta pesada executa por botão, não por mudança de filtro]] — rascunho e
  aplicado; o verbo do botão segue a ação real da tela.

Princípios: [[Todo estado da tela tem visual]] · [[Estado compartilhável mora na URL]]

## Checklist de tela nova

1. Cor e espaço saem de token e de escala — nada de hex ou `17px` solto.
2. Container com largura máxima, padding constante e gap por nível.
3. Hierarquia por superfície; borda só onde a superfície não resolveu.
4. Os quatro estados desenhados: carregando, vazio, erro, sucesso.
5. Filtro, busca e aba na URL; preferência pessoal no `localStorage`.
6. Testado no tema claro **e** no escuro, e no build de produção
   ([[Verificar no build de produção, não só em dev]]).

## Onde uma nota nova entra

Se fala de cor ou tema → **Cor e tema**. De medida, grade ou respiro → **Layout e
espaço**. De um bloco de UI concreto → **Componentes**. Do que a tela faz enquanto ou
depois de uma ação → **Estados e feedback**.

Se não couber em nenhuma e ainda assim for verdade sem CSS, é princípio: vai pra
[[Base]].

---

Voltar para [[Início]]
