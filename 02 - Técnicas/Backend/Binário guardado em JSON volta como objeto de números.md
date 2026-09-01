---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-09-01
---

# Binário guardado em JSON volta como objeto de números

> `Buffer` e `Uint8Array` não sobrevivem a `JSON.stringify`: viram
> `{"0":24,"1":91,...}`. Guardar chave criptográfica, credencial ou assinatura numa
> coluna `jsonb` sem serialização explícita grava algo que **parece** salvo e não é.

## O erro que evita

O defeito não aparece na hora. A sessão que gravou continua funcionando, porque a
chave viva está em memória — o que foi para o banco é só cópia. Ele aparece na
**subida seguinte**, quando o processo lê de volta e a biblioteca de criptografia
recebe um objeto onde esperava bytes.

E aparece mal: dependendo da biblioteca, o sintoma é uma desconexão genérica, uma
sessão "expirada", um handshake que falha sem dizer por quê. Ninguém olha para a
coluna `jsonb`, porque ela tem conteúdo.

## A costura

Um par de funções de ida e volta, aplicado em **todo** caminho de gravação e leitura:

```ts
const paraJson = (v: unknown) => JSON.parse(JSON.stringify(v, BufferJSON.replacer));
const deJson = <T,>(v: unknown): T => JSON.parse(JSON.stringify(v), BufferJSON.reviver) as T;
```

O `JSON.stringify(v)` do lado da leitura existe porque o driver já devolve o `jsonb`
como objeto: para aplicar o *reviver* é preciso voltar a ter texto. Bibliotecas que
esperam ser persistidas costumam trazer o par pronto (o Baileys traz o `BufferJSON`) —
procurar antes de escrever o seu.

## Como provar

Guardar, **abrir um segundo leitor do zero** e comparar bytes. Não basta ler de volta
no mesmo processo, que é onde o cache esconde o defeito:

```
ok  a chave privada volta como binário, não como objeto de números
ok  os bytes são idênticos
ok  a assinatura sobrevive
```

A primeira linha é a que importa: comparar tamanho ou conferir "não é nulo" passa nos
dois casos.

## O que mais vale lembrar

Credencial de sessão viva pertence ao banco, não a uma pasta do container. Em pasta
ela morre a cada redeploy, e recuperar significa uma pessoa com o celular na mão
refazendo o pareamento — ver
[[Estado desejado persistido religa o robô depois do restart]].

## Conexões
- Princípio: [[Configuração vem do ambiente, não do código]]
- Irmã: [[Estado desejado persistido religa o robô depois do restart]] · [[Numeric e bigint do Postgres chegam como string no driver pg]]
- Visto em: [[CRM Contábil]]
- Mapa: [[Backend]]
