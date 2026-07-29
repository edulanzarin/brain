---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-07-29
---

# Trocar o arquivo repede a senha e relê os dados

> Num formulário alimentado por um arquivo — a senha que o abre, os campos lidos
> de dentro dele — substituir o arquivo tem que recalcular tudo que derivava do
> antigo, na mesma ação. Herdar o estado velho (a senha, principalmente) salva um
> registro que não corresponde ao arquivo novo.

## A armadilha

O formulário carrega o arquivo e, ao lado, o estado que veio dele: a senha, os
campos extraídos. Trocar só o arquivo e deixar a senha antiga no campo parece
inofensivo — até o arquivo novo ter outra senha. Aí salva com a senha que não
abre o arquivo, e o registro fica inutilizável (ou a gravação falha).

Foi o que aconteceu no cofre ao renovar um certificado: o `.pfx` novo tinha senha
diferente, o form manteve a antiga carregada e salvou com ela.

## O padrão

Escolher o arquivo é o gatilho da leitura, não um passo separado. Ao
arrastar/selecionar o `.pfx`, abre um modalzinho que pede a senha e lê os dados na
hora (`parsePfx(arquivo, senha)`). O arquivo só é **adotado** — bytes, nome e
campos do form — depois que a leitura decifra. Some o botão "Ler dados" solto:
ler deixa de ser opcional.

O ganho não é só de fluxo: a senha que sobra no form é, **por construção**, a que
abre aquele arquivo. Confiar que o usuário atualize o campo de senha ao trocar o
arquivo é processo, e processo falha em silêncio; só adotar o arquivo depois que a
senha o decifra é estrutura — ver [[Um invariante se garante na estrutura, não no processo]].

O mesmo raciocínio vale pro caminho oposto: numa edição que **não** troca o
arquivo, não reenviar/regravar os bytes que não mudaram — o binário só sai de novo
pra rede quando de fato há um arquivo novo.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[Portal condicional dispensa o flag de montagem]]
- Visto em: [[Cofre Digital]]
- Mapa: [[Frontend]]
