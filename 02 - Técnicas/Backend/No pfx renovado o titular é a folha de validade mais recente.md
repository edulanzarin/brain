---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-07-29
---

# No .pfx renovado o titular é a folha de validade mais recente

> Um `.pfx`/PKCS#12 traz vários certificados (a cadeia da AC) e, quando é uma
> renovação, às vezes o **antigo (vencido) junto** — mesmo titular e emissor. Achar
> o "titular" pela primeira folha que aparece pega o que estiver na frente na ordem
> do arquivo, que pode ser o vencido. Entre as folhas, escolha a de **validade mais
> recente**.

## O problema

O titular de um `.pfx` é a folha da cadeia: o certificado que não assina nenhum
outro da lista (as ACs raiz/intermediária assinam; o dele não). O jeito direto é
`certs.find(folha)` — pega a primeira que passa no teste.

Mas "folha" não é único. Um `.pfx` renovado exportado do Windows costuma embutir o
certificado anterior, com **o mesmo titular e o mesmo emissor**. Aí existem duas
folhas (o renovado e o vencido) e `find` devolve a **primeira na ordem dos bags** —
ordem acidental do arquivo. Se o vencido vem primeiro, a leitura grava a data de
vencimento velha.

No Cofre Digital isso deixava o certificado marcado como "Vencido" na lista mesmo
depois de trocar pelo arquivo renovado: a data lida vinha do cert antigo embutido.
O status é calculado ao vivo do `expiresAt`, então bastava gravar a data errada uma
vez para a lista seguir vermelha. Difícil de ver, porque o arquivo estava certo — o
errado era **qual certificado dentro dele** a leitura escolhia.

## A solução

Filtrar todas as folhas e, entre elas, ficar com a de `notAfter` maior — o
certificado vigente, não o primeiro da lista:

```ts
const leaves = certs.filter((c) =>
  certs.every((other) => other === c || dn(other.issuer) !== dn(c.subject)),
);
const leaf = (leaves.length ? leaves : certs).reduce((best, c) =>
  c.validity.notAfter > best.validity.notAfter ? c : best,
);
```

O `filter` das folhas roda **antes** do `max`: uma AC costuma ter validade bem mais
longa (10, 20 anos) que a folha, então escolher o maior `notAfter` no conjunto
inteiro pegaria a AC. Descartar quem assina outro cert primeiro, desempatar por
validade depois.

## O que mais vale lembrar

O defeito não era ler errado — era **confiar na ordem** em que os certificados
aparecem no arquivo pra decidir qual é o certo. Ordem de coleção é propriedade
acidental; quando mais de um item passa no critério, desempate por algo
**significativo** (aqui, a validade), não por quem chegou primeiro. Mesmo espírito
de [[Um invariante se garante na estrutura, não no processo]]: a garantia mora numa
regra explícita, não num "costuma vir na ordem certa".

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[Trocar o arquivo repede a senha e relê os dados]]
- Visto em: [[Cofre Digital]]
- Mapa: [[Backend]]
