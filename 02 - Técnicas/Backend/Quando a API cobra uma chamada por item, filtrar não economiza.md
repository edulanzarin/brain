---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-28
---

# Quando a API cobra uma chamada por item, filtrar não economiza

> Antes de estreitar um filtro para "baixar menos", identifique o que a API
> cobra. Onde o recurso exige um identificador no caminho, o custo é **uma
> chamada por entidade** — e filtro nenhum muda esse número. Filtrar reduz o
> payload; o teto de requisições continua exatamente onde estava.

## O erro que parece otimização

Pedir só o setor que interessa soa econômico, e é — do lado do payload. Mas se o
endpoint não aceita "liste tudo" e cobra o identificador no caminho, varrer a
carteira já custa N chamadas, uma por empresa, com filtro ou sem. Estreitar o
recorte deixa a resposta menor e o **tempo de varredura idêntico**, porque quem
manda é o teto de requisições por minuto, não os bytes.

O preço de acreditar no contrário é alto ao contrário: você restringe o escopo do
produto ("por enquanto só o Contábil") para poupar um custo que não existe, e aí
paga a varredura inteira de novo quando a segunda área quiser a dela.

## A regra

**Meça o que domina antes de otimizar o que não domina**
([[Fator que domina o resultado não entra na conta por estimativa]]). Se o
gargalo é o número de chamadas, traga TODAS as dimensões de uma vez e recorte na
leitura: o custo é o mesmo e as áreas seguintes entram de graça.

Filtrar na origem volta a valer quando o filtro muda o número de chamadas — uma
lista paginada em que o recorte encurta a paginação, por exemplo — ou quando a
resposta é grande a ponto de o tempo de transferência competir com o
espaçamento entre chamadas.

## Sinal de que você está neste caso

O recurso quer um id no caminho (`/recurso/{cnpj}`), o "listar todos" devolve
vazio, e a documentação fala de filtros como se fossem economia. Aí filtro é
conveniência de leitura, não plano de capacidade.

## Conexões
- Princípio: [[Fator que domina o resultado não entra na conta por estimativa]]
- Irmã: [[Chamada externa tem timeout e erro tratado]] · [[Recusa não é falha: contra o não do servidor, insistir é ruído]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Backend]]
