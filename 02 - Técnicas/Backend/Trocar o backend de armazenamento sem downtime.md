---
tags: [tipo/atomica, camada/padrao, dev/backend]
criado: 2026-07-22
---

# Trocar o backend de armazenamento sem downtime

> Mover binários de dentro do banco para uma pasta (ou entre dois backends
> quaisquer) com uma coluna de ponteiro, leitura com reserva e migração sob
> demanda — sem parar o app nem arriscar o que já existe.

## O problema

Os arquivos (um .pfx, um PDF, uma imagem) estavam como base64 no Postgres e
precisavam ir para uma pasta na rede. Reescrever tudo de uma vez e apagar as
colunas seria um corte seco: qualquer arquivo que não migrasse certo some. E
enquanto a migração roda, o app tem que continuar servindo os dois lados.

## A solução

Uma coluna de ponteiro por registro (`filePath`) ao lado da coluna antiga
(`fileData`/base64), e três costuras — é a aplicação de
[[Migração de dados mantém o antigo como reserva até a virada]]:

- **Gravar**: se há pasta configurada, escreve em disco (`<raiz>/<cat>/<id>.<ext>`,
  o `id` é uuid nosso, nunca nome do usuário) e guarda só o `filePath`; senão,
  mantém no banco. Sem pasta, nada muda.

```ts
// disco primeiro, banco como reserva
async function loadBytes(filePath: string | null, dbBase64: string | null) {
  if (filePath) { const b = await readFileAt(filePath); if (b) return b; }
  return dbBase64 ? Buffer.from(dbBase64, "base64") : null;
}
```

- **Ler**: `loadBytes` tenta o disco e cai pro banco — o download e o form de
  edição enxergam o arquivo do mesmo jeito, esteja onde estiver.
- **Migrar**: endpoint sob demanda (admin) que, para cada registro ainda no banco,
  grava em disco e **só então** zera o base64. Seguro repetir, seguro parar no
  meio (quem já tem `filePath` é pulado).

## O que mais vale lembrar

- **Editar sem reenviar o arquivo não pode apagá-lo.** Se o form manda o campo de
  arquivo vazio num registro que tem arquivo, preserva o que existe em vez de
  limpar — senão um disco fora do ar na hora da edição apaga a referência. Só
  limpa quando há sinal explícito de "sem arquivo" (no certificado, virar cartão).
- **Resolver o caminho dentro da raiz** barrando `..`, para o ponteiro do banco
  nunca escapar da pasta.
- **A pasta é configuração, a senha de trocá-la é segredo de ambiente** — não vão
  pro banco nem pro cliente. Ver [[Configuração vem do ambiente, não do código]].
- Irmã da rota que serve o arquivo já protegido:
  [[Servir anexo por rota com checagem de permissão]].

## Conexões
- Princípio: [[Migração de dados mantém o antigo como reserva até a virada]]
- Irmã: [[Servir anexo por rota com checagem de permissão]]
- Visto em: [[Cofre Digital]]
- Mapa: [[Backend]]
