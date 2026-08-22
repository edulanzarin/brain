---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-22
---

# Sonda que falhou não é sinal de que mudou

> Cache barato quase sempre tem uma **sonda**: uma pergunta minúscula — `HEAD`, `ETag`,
> `Last-Modified`, `count`, `updated_at` — que decide se vale buscar o volume inteiro.
> A sonda tem três respostas possíveis, e quase toda implementação só prevê duas.

## A regra

O retorno da sonda é **tri-estado**, não booleano:

- `igual` — segura o que está em memória;
- `mudou` — busca o volume;
- `desconhecido` — **não busca**, segura o que está em memória e **espaça** a próxima
  pergunta.

O erro clássico é `catch { return null }` e o chamador lendo `null` como "mudou". Aí
qualquer falha da sonda — DNS, TLS, timeout, `405` — vira download completo. E o modo
degradado é silencioso: nada quebra, nada aparece na tela, o dado continua certo. O
único sintoma mora na conta de banda do **outro**.

Cenário concreto: uma CDN passar a recusar `HEAD` (`405`/`403` é comum e está fora do
seu controle) desliga o mecanismo de `ETag` inteiro sem uma linha de log — e o site passa
a baixar ~1,6 MB a cada janela de 10s, para sempre.

## Três consequências que andam juntas

**Log por transição, não por ocorrência.** Degradação silenciosa precisa de rastro, mas
um aviso a cada 10s vira ruído que ninguém lê. Guarde o estado anterior e avise só quando
ele muda — inclusive na volta.

**Cadência é variável própria, não `checado + janela`.** Existem pelo menos três ritmos
(normal, sonda cega, fonte caída), e derivá-los de um campo de dado mistura "quando
perguntei" com "quando vale perguntar de novo". Um `reperguntarEm` explícito separa os
dois.

**Falha precisa ser gravada.** Se o ramo de erro devolve sem escrever no estado
guardado, o carimbo de tempo congela e a trava de frescor nunca mais engata: toda
requisição seguinte reentra e paga o timeout inteiro. Com a fonte fora do ar, isso é a
página inteira travando por dezenas de segundos **por visita**.

## Aceitar a resposta também precisa de piso

Validar o volume com "array não vazio" aceita 3 registros do mesmo jeito que 482. Um
arquivo publicado pela metade passa, invalida toda derivação memoizada e o sistema serve
o catálogo truncado com selo de dado vivo. A régua barata é o **snapshot versionado**:
patch legítimo nunca corta metade do conjunto, arquivo truncado sempre corta.

E dado antigo da **fonte real** ganha do snapshot do build sempre — quem decide é a
procedência do dado, não o selo que ele carrega no momento.

## Conexões
- Princípio: [[Peça o que a fonte mostra, não o que você precisa]] ·
  [[Estado mutável se lê da fonte no uso, não de cópia guardada]]
- Irmã: [[Travar o valor não impede a tela de afirmar a partir dele]]
- Visto em: [[piwdex2]]
- Mapa: [[Backend]]
