---
tags: [tipo/atomica, camada/padrao, dev/backend]
criado: 2026-08-30
---

# Artefato prova existência; só um desafio prova o momento

> Pedir "mande uma foto sua" ou "mande um vídeo seu" não verifica nada além de
> que aquele arquivo existe. Ele pode ter sido gravado ontem, há três anos, ou
> por outra pessoa. O que transforma o arquivo em prova é a casa **sortear algo
> antes** e exigir que apareça dentro dele.

## A forma

1. O sistema sorteia um valor que ninguém tinha como saber (um código curto, um
   número, uma frase).
2. Ele é entregue à pessoa **com prazo**.
3. A prova só vale se o valor sorteado estiver dentro dela e o prazo não tiver
   vencido.

O prazo não é burocracia — é a metade do mecanismo. Um código eterno vira
apenas mais uma coisa a copiar: quem quiser fraudar pede o código, grava uma vez
e reusa a gravação em todas as contas que abrir. Com prazo, cada prova custa uma
gravação nova.

É o mesmo desenho do registro TXT que um provedor de DNS pede para confirmar que
o domínio é seu, e do código que o banco manda antes da transferência. Muda o
que se prova; a forma é uma só.

## O que isso NÃO prova

Que a pessoa quis. Um código na mão prova frescor e posse do canal, não
consentimento livre — quem está sendo coagido também consegue segurar o papel.
Verificação de identidade resolve fraude de identidade, e nada mais; tratar o
selo como prova de que está tudo bem é a leitura que ele não sustenta.

## Detalhes que decidem se funciona

- **O valor tem que aparecer no artefato**, não ao lado dele. Código digitado
  num campo enquanto se envia um vídeo antigo não prova nada: os dois chegam
  juntos e nenhum depende do outro.
- **Pedir de novo com um código válido na mão não sorteia outro.** Quem já
  escreveu o papel e clicou por engano perderia o trabalho, e ficaria com dois
  códigos sem saber qual vale. Só o vencido é substituído.
- **O prazo se apura na hora de usar**, comparando com agora — não por agendador
  que marca o vencido. Ver [[Rotação por período se apura na leitura, e dispensa agendador]].

## Conexões
- Princípio: nenhum cobre ainda — folha isolada. **Candidato a princípio**:
  "existência e momento são afirmações diferentes, e artefato só prova a
  primeira" deve reaparecer fora de verificação de pessoa (confirmação de
  domínio, prova de posse de chave, anti-replay). Na segunda aparição, promover
  para [[Base]].
- Irmã: [[Código que a pessoa copia à mão não pode ter caractere ambíguo]]
- Irmã: [[Registro com estado não se confere pela existência]]
- Visto em: [[Privello]]
- Mapa: [[Backend]]
