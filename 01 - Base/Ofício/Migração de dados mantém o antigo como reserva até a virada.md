---
tags: [tipo/atomica, camada/principio, dev/backend]
criado: 2026-07-22
---

# Migração de dados mantém o antigo como reserva até a virada

> Trocar onde um dado mora é aditivo: grava no destino novo, lê do novo com o
> antigo de reserva, e só descarta o antigo depois de a virada estar confirmada.

## A regra

Mudar o armazenamento de um dado (de coluna para arquivo, de um volume para
outro, de um banco para outro) nunca é um corte seco. É uma sequência:

1. **O novo caminho passa a receber**, sem exigir que o antigo já esteja vazio.
2. **A leitura tenta o novo e cai pro antigo** enquanto houver dado dos dois lados
   — nada quebra no meio da transição.
3. **A migração do que ficou pra trás roda sob demanda**, é segura de repetir e
   de parar no meio: cada registro só perde a cópia antiga *depois* de a nova
   estar gravada.
4. **O antigo só é descartado quando a virada está confirmada** — nunca no mesmo
   gesto que cria o novo.

Quem apaga a origem antes de confirmar o destino transforma uma migração numa
aposta: qualquer falha no meio vira perda.

## Por que

Migração é onde o dado está mais exposto: existe em dois lugares, meio convertido,
e um erro não tem de onde voltar. Manter o antigo como rede transforma "deu erro,
perdi" em "deu erro, tento de novo". O custo é conviver um tempo com dado
duplicado; o benefício é que nenhuma etapa é irreversível sozinha.

No Cofre Digital isso apareceu duas vezes. Ao mover o volume do Postgres para o
padrão de nome novo, fez-se `pg_dump` antes e o volume antigo ficou parado como
segurança. Ao tirar os arquivos (.pfx, PDF, imagem) de dentro do banco para uma
pasta, a gravação nova vai pro disco mas a leitura é disco-primeiro-banco-reserva,
e o migrador só zera o base64 depois de gravar o arquivo — ver
[[Trocar o backend de armazenamento sem downtime]].

## Na prática

- A virada é **opt-in**: enquanto o destino novo não está configurado, tudo segue
  no antigo. O recurso não pode quebrar quem ainda não migrou.
- Um teste que exercita a migração nunca aponta o destino para um lugar
  descartável com dado real dentro — é assim que se perde o que se queria mover.
  Casa com [[Semear teste cria linha nova, não muta linha real]].

## Conexões
- Irmã: [[Semear teste cria linha nova, não muta linha real]]
- Técnica que aplica: [[Trocar o backend de armazenamento sem downtime]]
- Visto em: [[Cofre Digital]]
- Mapa: [[Base]]
