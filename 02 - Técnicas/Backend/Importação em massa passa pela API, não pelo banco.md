---
tags: [tipo/atomica, camada/padrao, dev/backend]
criado: 2026-07-29
---

# Importação em massa passa pela API, não pelo banco

> Migrar um monte de registros para um sistema é tentador de fazer com INSERT
> direto no banco — é rápido e "é só dado". Mas aí a carga não passa por nenhuma
> das regras que o app garante no cadastro normal: validação, resolução de dono,
> deduplicação, gravação de arquivo. Manda os registros pelo MESMO endpoint que
> uma pessoa usaria; a importação herda todas as guardas de graça.

## O problema

O INSERT direto parece limpo, mas cada regra que vive na rota de cadastro tem que
ser reimplementada na mão na carga — e as duas divergem. O resultado é dado que o
app nunca teria aceitado: documento inválido, sem empresa dona, arquivo não
gravado no lugar certo, duplicado. É o mesmo buraco de
[[Criar e editar passam pelo mesmo funil de resolução]], agora com uma TERCEIRA
porta de escrita (a migração) fugindo do funil.

## A solução

O importador é um cliente HTTP: loga como um usuário e faz `POST` no endpoint de
cadastro, um por registro. Ganha sem reescrever nada:

- **Validação** idêntica à da tela (no cofre: dígito verificador do CNPJ, tipo x
  documento, datas) — [[Dígito verificador rejeita o documento errado na entrada]].
- **Resolução de dono/vínculo**: a empresa nasce pelo CNPJ, o grupo é criado e
  anexado — a mesma regra do `resolveCertCompany`.
- **Idempotência**: o anti-duplicata da rota faz reexecutar não duplicar; a carga
  pode parar no meio e recomeçar.
- **Efeitos colaterais** (gravar o `.pfx` na pasta certa) acontecem sozinhos.

Fica sequencial/concorrência baixa, com relatório de sucesso/falha por item.

Dois detalhes de migração que se pagaram:

- **Quando não há acesso ao banco antigo, a API dele é o exportador.** O sistema
  velho servia tudo em `GET /api/certificados` (senha inclusa) — coleta num
  request só, sem tocar no banco que ninguém queria arriscar.
- **O arquivo é a fonte de verdade dos metadados; o registro antigo só guarda o
  ponteiro + o segredo.** O `.pfx` tem titular, AC, emissão e vencimento; o banco
  antigo só tinha o caminho e a senha. A ponte é uma **chave natural** no nome do
  arquivo (`NOME - CNPJ.pfx`), conferida como única antes de confiar nela.

## O que mais vale lembrar

Vale quando o volume é grande mas cabe em requests (dezenas a milhares), não
milhões — aí o custo por request pesa e o caminho é carga no banco com a
validação replicada de propósito. E exige um jeito de autenticar o importador
(sessão de admin), o que é justo: a migração é uma escrita como outra qualquer.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[Criar e editar passam pelo mesmo funil de resolução]]
- Irmã: [[Dígito verificador rejeita o documento errado na entrada]]
- Visto em: [[Cofre Digital]]
- Mapa: [[Backend]]
