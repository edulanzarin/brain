---
tags: [tipo/projeto, projeto/poke-idle]
criado: 2026-08-25
status: ativo
codigo_em: ~/Dev/poke-idle
---

# Vespéria

> Idle de Pokémon no navegador, 2D top-down, pixel art. Você é um treinador em
> Vespéria, manda equipes pras rotas do vale, elas caçam sozinhas e você volta
> pra decidir o que fazer com o que trouxeram. Resposta ao Poke Idle World, mas
> pela mecânica: o que se pega dele é o dado e a regra do gênero; a cidade, os
> sistemas e o texto são nossos.

Código em: `~/Dev/poke-idle` · app **4073**, banco **5073**
GDD: `docs/game-design.md` (fonte da verdade do design)

## O que o diferencia

Cinco pilares, e os cinco existem porque o jogo de referência não os tem:

1. **A rota é população viva.** Cada espécie tem abundância que cai ao caçar e
   volta com o tempo, em crescimento logístico com imigração e acoplamento
   predador-presa. Doze horas de caçada derrubam a rota pra 48%; doze de descanso
   devolvem 99%. Consequência: não existe "a melhor rota", existe a melhor rota
   agora, e alternar vira estratégia em vez de conselho no manual.
2. **Captura por Vínculo.** Conhecimento por espécie sobe com encontro e melhora
   a chance; um contador de tentativas garante a captura ao cruzar o limiar. Os
   dois números vão pra tela. O gênero esconde o medidor e o jogador conclui que
   a sorte quebrou.
3. **Expedição tem ordens**: doutrina (caçada/captura/coleta/treino) e risco
   (margem/interior/coração), cada um com preço medido.
4. **Hora e clima de relógio real** mudam quem aparece, o combate e a captura.
5. **A cidade é mapa, não menu**: 160x120 tiles, cada sistema é um lugar.

## Estado atual

**Jogável de ponta a ponta (ago/2026).** Andar por Vespéria, montar time, mandar
pra rota, voltar e ler o relatório. Quatro cortes na `main`.

- **Motor puro** em `src/engine`: sem React, sem Prisma, sem `Date.now()` nem
  `Math.random()` solto. O tempo entra por parâmetro e o acaso por PRNG semeado,
  então online e offline usam o mesmo caminho — é o
  [[Progresso idle é função pura do tempo semeada, não simulação tick a tick]]
  aplicado inteiro. 28 testes.
- **Servidor autoritativo** em Postgres + Prisma 7. Sessão opaca em cookie
  httpOnly, scrypt com comparação de tempo constante, compra debitada condicionada
  ao saldo. A regeneração da rota é preguiçosa: integra o Δt na leitura, sem job
  de fundo.
- **Cliente PixiJS**: chão recortado por câmera (19 200 tiles, pool só do
  visível), peça e ator no mesmo container ordenado por Y — o jogador passa atrás
  do telhado sem camada extra.
- **Arte própria** pra cidade, em Python com Pillow, paleta fechada. As sprites
  de Pokémon vêm do [[PMDCollab SpriteCollab - sprites de Pokemon em 8 direcoes]]:
  968 espécies andando em oito direções.
- **Dado de build**: 1025 espécies e 505 golpes da PokeAPI, materializados em
  JSON. Em runtime o jogo não fala com ninguém de fora.

## Decisões que valem lembrar

- **Ecologia é por treinador, não global.** Mundo compartilhado é tentador e
  vira grieffing: um jogador esgota a rota de todos. Fica no roadmap, e mexe só
  numa tabela.
- **Nome e cidade são constante**, em `src/engine/constants.ts`. Trocar é uma
  linha.
- **Rotas e cidade são dado, não código.** Rota nova é uma entrada em
  `routes.ts`; bairro novo é uma função no gerador do mapa.

## O que quebrou e ensinou

- Um Charmander sozinho morria em 106 segundos na rota inicial. A calibração fora
  feita com time de seis, e sozinho ele toma o dano inteiro num combate seis
  vezes mais longo → [[Calibre nas pontas, o meio esconde o defeito]].
- Varrer a taxa de recuperação em três valores dava sempre o mesmo veredito, só
  mudando o quando. A faixa do meio só apareceu com auto-cura por consumível →
  [[Desgaste constante contra recuperação constante é bimodal]].
- Um Pidgey de nível 5 derrubava um time de nível 20 porque o multiplicador de
  tipo entrava dentro da razão de mitigação →
  [[Multiplicador de contexto entra depois da razão, não dentro dela]].
- A tela ficava na página de erro do Next em dev, e não no código: o Pixi era
  destruído antes de terminar o init →
  [[Efeito que roda duas vezes destrói o que ainda não terminou de nascer]].
- O mosaico da praça virou poá porque o desenho não tocava a borda do tile →
  [[Motivo de piso tem que tocar a borda do tile]].

Os três primeiros só apareceram **jogando**, não lendo: dois numa sonda de
balanceamento que roda o motor e imprime a tabela, um num navegador headless
tirando foto da tela.

## Próximos passos

Roadmap por fatia vertical, no GDD §10. As duas seguintes:

- **Progressão**: evolução (o catálogo já traz a cadeia), treino dirigido e uso
  de item fora da expedição.
- **Endgame**: Bolsa entre treinadores, Arena e chefe de rota.

Fora do roadmap, anotado: o painel do Portão pede uma projeção de rendimento por
hora antes de enviar — o motor já tem `previewRate` pronto pra isso.

## Conexões
- Usa: [[Infra]] · [[Backend]] · [[Frontend]] · [[Design]]
- Referência: [[PMDCollab SpriteCollab - sprites de Pokemon em 8 direcoes]] ·
  [[Poke Idle World - endpoints publicos de dados]]
- Princípios: [[Progresso idle é função pura do tempo semeada, não simulação tick a tick]] ·
  [[Calibre nas pontas, o meio esconde o defeito]] ·
  [[Desgaste constante contra recuperação constante é bimodal]]
- Técnicas: [[Multiplicador de contexto entra depois da razão, não dentro dela]] ·
  [[Efeito que roda duas vezes destrói o que ainda não terminou de nascer]] ·
  [[Motivo de piso tem que tocar a borda do tile]] ·
  [[Mundo imperativo e React se falam por eventos, não por referência]]
- Mapa: [[Projetos]]
