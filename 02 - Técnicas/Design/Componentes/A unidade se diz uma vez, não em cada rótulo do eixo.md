---
tags: [tipo/atomica, camada/padrao, design]
criado: 2026-09-02
---

# A unidade se diz uma vez, não em cada rótulo do eixo

> Reusar o formatador de dinheiro no eixo do gráfico parece economia e é
> defeito: "R$ 8,0 mi" repetido cinco vezes não cabe na canaleta, **quebra em
> duas linhas** e a segunda linha invade a área do dado.

## O problema

O mesmo `dinheiroCompacto` servia à dica no apontador e ao rótulo do eixo. Na
dica está certo — ali o valor aparece sozinho e precisa se identificar. No eixo,
o símbolo da moeda aparece uma vez por marca, rouba largura de um espaço que já
é apertado, e quando não cabe o rótulo enrola por cima da série.

E é informação que ninguém precisava: o cartão já diz "Entradas e saídas, **em
valor**" no subtítulo. O eixo estava repetindo cinco vezes o que o título disse
uma.

## A solução

Dois formatadores, com trabalhos diferentes:

| Onde | Formato | Por quê |
|---|---|---|
| Dica, legenda, célula | `R$ 4.281.930,22` | valor aparece sozinho, tem que se identificar |
| Rótulo de eixo | `4,3 mi` | a unidade já está no título; aqui só a magnitude |

O componente expõe as duas (`formatar` e `formatarEixo`), com o terso como
padrão — assim o caminho fácil é o certo, e quem quiser moeda no eixo precisa
pedir.

## O que mais vale lembrar

- **Rótulo de categoria também quebra.** Nome de empresa é sempre longo, e a
  biblioteca de gráfico quebra em duas linhas em vez de cortar — a segunda linha
  colide com a barra vizinha. O corte é explícito, com reticências, e a dica no
  apontador entrega o nome inteiro.
- O defeito só aparece **renderizado**. Ele não é erro de tipo, não é aviso de
  build e não aparece com dados pequenos; nasce quando o valor cresce o
  bastante para o rótulo esbarrar na largura da canaleta.

## Conexões
- Princípio: [[Nota carrega só o que a pessoa não sabe]]
- Irmã: [[Blocos de dado - card, KPI e gráfico]] · [[Ponto decimal em interface pt-BR afirma outro número]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Design]]
