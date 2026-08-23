---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-24
---

# Freio de oscilação vale para a máquina, não para a ordem de quem manda

> Sistema que decide sozinho precisa de freio: sem um piso de tempo entre
> trocas, dois candidatos que empatam fazem ele alternar a cada leitura. O freio
> costuma nascer como um `if (agora - ultimaTroca < X) return` no topo da rotina
> de decisão — e é aí que ele passa a frear também **quem mandou**.

## O sintoma

No [[piwdex2]] o robô escolhe onde caçar por objetivo. O freio é de 60s, e a
leitura que dispara a decisão chega a cada 20s. Trocar de objetivo na tela podia
levar **um minuto e meio** para mudar qualquer coisa visível.

Ninguém lê isso como "o freio está funcionando". Lê-se como *não fez nada* — e a
reação natural é clicar de novo, que também não faz nada, o que confirma a
conclusão errada.

## A distinção

O freio protege contra **indecisão da máquina**: dado que empata, ruído de
medição, fronteira entre duas faixas. Uma ordem explícita não é nada disso — ela
é a informação nova mais confiável que o sistema vai receber.

```ts
usarConfig(cfg) {
  const mudouObjetivo = cfg.objetivo !== this.cfg.objetivo /* ... */;
  this.cfg = cfg;
  if (mudouObjetivo) {
    this.ultimaTroca = 0;   // a ordem não espera o freio da máquina
    this.decidirAgora();    // e nem a próxima varredura
  }
}
```

- **Zere o relógio, não pule o freio.** Zerar mantém a proteção contra a
  oscilação seguinte; um `if (forcado)` espalhado pela rotina vira exceção que
  ninguém mais entende seis meses depois.
- **Aja na hora, não na próxima varredura.** Guardar a intenção e esperar o
  ciclo soma a latência do freio à do poll — e as duas somadas é que produzem o
  minuto e meio.
- **A mesma regra vale para cache e debounce**: o intervalo existe para conter
  repetição automática, e clique não é repetição automática.

## O que mais vale lembrar

Quando um controle parece não responder, o suspeito raro é o bug e o comum é uma
**salvaguarda aplicada larga demais**. Ela foi escrita para um caso, ficou no
caminho de todos, e não deixa rastro em log nenhum — porque, do ponto de vista
dela, nada aconteceu.

## Conexões
- Princípio: [[Estado mutável se lê da fonte no uso, não de cópia guardada]]
- Irmã: [[Objetivo é exclusivo, interruptor é combinável]] ·
  [[Config que o motor executa mora no servidor e se aplica em todo início de fluxo]] ·
  [[Comando sem resposta precisa de vigia, não de fé]]
- Visto em: [[piwdex2]]
- Mapa: [[Backend]]
