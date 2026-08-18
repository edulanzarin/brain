---
tags: [tipo/atomica, camada/principio, design]
criado: 2026-08-18
---

# Dado que chega preenche espaço reservado, não empurra a tela

> O layout é definido pelo **esquema do que pode aparecer**, não pelo que está
> presente agora. Se um valor pode existir, o lugar dele já existe antes do valor.

## O quê

Interface que se reorganiza quando o dado chega transfere o custo da variação
para quem lê: o olho perde a âncora, o dedo erra o botão que se moveu, a barra
fixa muda de altura e a página inteira desce. A regra é reservar: todo dado
dinâmico tem um lugar permanente, e a chegada (ou sumiço) do valor troca apenas
o **conteúdo** desse lugar — nunca a geometria em volta.

Vale em qualquer tecnologia: numa tela web, num terminal, num relatório que se
regenera. `{valor && <bloco>}` é a forma mais comum de violar — o bloco inteiro
entra e sai do fluxo ao sabor do dado.

A exceção é a **escolha do usuário**: trocar de aba, abrir um modal, expandir um
acordeão pode reestruturar a tela, porque quem pediu a mudança foi quem está
olhando. O proibido é a tela mudar sozinha porque um número chegou do servidor.

## Por que importa

Visto duas vezes, em formas diferentes do mesmo erro:

- No [[Cofre Digital]], listas carregando eram `<div>` vazio: a página nascia
  curta e **crescia** quando os dados chegavam. A resposta foi o esqueleto com a
  forma do conteúdo — a versão "carregando" deste princípio.
- No [[piwdex]], o HUD fixo da área VIP era um `flex-wrap` com nove blocos
  condicionais: cada dado do robô que chegava pelo stream mudava a altura da
  barra **sticky** e deslocava a página inteira, várias vezes por minuto. A
  resposta foi o slot permanente com placeholder — a versão "dado vivo".

Carregamento e tempo real são o mesmo problema: um acontece uma vez, o outro
para sempre.

## Conexões
- Irmã: [[Todo estado da tela tem visual]] · [[Escala fechada em vez de valor solto]]
- Exemplo: [[Esqueleto de carregamento imita a forma do conteúdo]] · [[Slot com placeholder esmaecido segura o lugar do dado vivo]]
- Mapa: [[Design]]
