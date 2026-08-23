---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-08-23
---

# Peso de página se mede no fio, não na saída do render

> `curl | wc -c` devolve o tamanho **bruto**. O que sai do servidor vem
> comprimido, e markup repetitivo comprime perto de 95%. A diferença entre os dois
> números é a diferença entre um problema grave e nenhum problema.

## O erro, com número

Numa auditoria de SEO do [[piwdex2]] eu registrei como falha de severidade ALTA:

    /dex serve 1,14 MB de HTML

Medindo o que de fato atravessa a rede:

| página | bruto | no fio (Brotli) | compressão |
|---|---|---|---|
| `/dex` (24 cards) | 1,14 MB | **68 KB** | 94% |
| `/dex/tipo/fogo` (42 cards) | 1,00 MB | **41 KB** | 96% |
| `/dex/1` (ficha) | 174 KB | **22 KB** | 87% |

68 KB para uma lista inteira é bom. Não havia nada pra consertar, e eu quase
gastei um dia inteiro reescrevendo componente pra resolver um problema
inexistente.

## Por que a diferença é tão grande justamente aqui

Compressão vive de repetição, e grade de card é a coisa mais repetitiva que uma
página tem: quarenta e dois blocos com a mesma árvore de classes, os mesmos
nomes de atributo, a mesma estrutura de barra de stat. O dicionário do Brotli
resolve o segundo card e todos os quarenta seguintes saem quase de graça.

O corolário incomoda: **quanto mais repetitivo o markup, menos vale otimizar o
markup.** A intuição de "tem HTML demais" está lendo o pior indicador possível.

## Como medir

```bash
curl -s -o /dev/null -w "bruto %{size_download}\n" \
     -H "Accept-Encoding: identity" https://exemplo/pagina
curl -s -o /dev/null -w "no fio %{size_download}\n" \
     -H "Accept-Encoding: br, gzip" https://exemplo/pagina
```

Sem o `Accept-Encoding: identity` na primeira, o `curl` pode já receber
comprimido e os dois números saem iguais — e aí a medição não mede nada.

## O que o bruto ainda custa (e é outro assunto)

Ele não some do problema, só muda de lugar: o navegador ainda **analisa** o
documento inteiro depois de descomprimir. Em celular fraco isso aparece como
tempo de renderização e de interação, não como tempo de download.

Então a regra é escolher o número pela pergunta:

- "está pesado pra baixar?" → **no fio**, e é quase sempre esta a pergunta;
- "está pesado pra renderizar?" → bruto, número de nós, e o teste é um aparelho
  ruim de verdade, não uma régua de bytes.

Confundir os dois é o que faz alguém reescrever componente achando que resolve
rede.

## Conexões
- Princípio: [[Verificar no build de produção, não só em dev]] ·
  [[Ordene pela grandeza que decide, não pela que impressiona]]
- Irmã: [[Animação de enfeite escolhe a propriedade pelo custo, não pelo efeito]]
- Visto em: [[piwdex2]]
- Mapa: [[Frontend]]
