---
tags: [tipo/atomica, camada/padrao, dev/backend, design]
criado: 2026-08-24
---

# Objetivo é exclusivo, interruptor é combinável

> Automação nasce como interruptor: "repor bolas", "vender drop", "subir de
> nível". Ligar dois é natural, e quase sempre funciona — até dois deles
> quererem mandar na **mesma** coisa. Aí não há prioridade que resolva: eles
> alternam a cada varredura, e o sistema parece indeciso porque ele é.

## O corte

A pergunta que separa os dois é **qual recurso a automação comanda**:

- **Interruptor** — comanda um recurso próprio. "Repor bolas" mexe na carteira,
  "vender drop" mexe na mochila. Combinam à vontade, e a tela pode ser uma lista
  de chaves.
- **Objetivo** — comanda um recurso *disputado*. No [[piwdex2]], "farmar
  dinheiro" e "subir o Golem até 200" decidem os mesmos dois valores: quem é o
  líder e em que campo ele está. Um valor, um dono.

Objetivo, então, não é chave: é **escolha entre alternativas**. Na interface isso
é um segmentado ou um rádio, e no modelo é uma união de strings, não N booleanos:

```ts
objetivo: "nenhum" | "dolares" | "nivel";
```

O tipo já impede o estado impossível. Com `autoDolares: boolean` e
`autoNivel: boolean`, o par `true/true` existe, alguém liga, e a partir daí o
código passa a precisar de uma regra de desempate que ninguém pediu.

## O que mais vale lembrar

- **"nenhum" é um objetivo**, e o mais importante: é o modo em que a máquina não
  decide nada por conta. Ele precisa estar na lista, não ser a ausência dela.
- **Anti-oscilação é obrigatório** mesmo com um objetivo só. A recomendação muda
  quando o dado muda, e dado que empata na fronteira faz o sistema trocar de
  alvo a cada leitura. Um piso de tempo entre trocas custa uma variável.
- **Mostre a recomendação antes de segui-la.** O objetivo é a máquina decidindo
  no lugar de alguém; ver o que ela decidiu *antes* de ela agir é o que separa
  delegar de perder o controle.
- O mesmo corte aparece fora de automação: modo de ordenação, estratégia de
  cache, política de retry. Sempre que duas opções escrevem no mesmo lugar, elas
  eram uma escolha e não duas chaves.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[Adquirir o recurso exclusivo é uma ação, usá-lo é outra]] (o mesmo
  recurso disputado, um nível acima: lá a disputa é pela sessão, aqui pelo
  comando que sai por ela) · [[Estado desejado persistido religa o robô depois do restart]]
- Visto em: [[piwdex2]]
- Mapa: [[Backend]]
