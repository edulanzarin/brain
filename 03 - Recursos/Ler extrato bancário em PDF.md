---
tags: [tipo/atomica, dev/backend, projeto/questor-bi]
criado: 2026-07-19
---

# Ler extrato bancário em PDF

> Quase todo extrato bancário brasileiro tem a MESMA forma — uma linha por lançamento começando com a data — e o que muda entre bancos é só como o sinal é indicado. Isso permite um motor único configurado por banco em vez de um parser por banco. E quando o extrato traz saldo corrente, `valor = saldo − saldoAnterior` dá o sinal de graça E valida a leitura inteira.

## O truque que sustenta tudo: a cadeia de saldos

A maioria dos extratos traz o **saldo depois de cada lançamento**. Então:

- **O sinal sai de graça**: `valor = saldo[n] − saldo[n−1]`. Não importa se o banco usa `-1.234,56`, `- 1.234,56`, `1.234,56 D` ou colunas separadas de débito/crédito.
- **A leitura se autovalida**: `saldoInicial + Σ lançamentos == saldoFinal`. Se fechar, quase certamente nenhuma linha foi perdida nem duplicada. É o equivalente de ter um gabarito sem ter o gabarito.

Sem OFX para comparar, essa é a única conferência objetiva disponível. Vale muito exibi-la na tela como sinal de confiança, em vez de o usuário confiar no silêncio.

## Os modos de sinal que aparecem na prática

| Modo | Exemplo | Bancos vistos |
|---|---|---|
| saldo corrente | `… 3.696,11  3.846,67` | Sicredi, Viacredi, Sifra |
| menos explícito | `-R$ 9,00`, `- 5.078,43` | Itaú, Daycoval, Bradesco, Belluno |
| sufixo C/D | `24.423,66 D`, `0,00C` | C6, Sicoob |

Detalhes que quebram parser ingênuo:

- **Menos separado do número** por espaços (é marcador de coluna de débito).
- **Data sem ano** (Daycoval usa `22/06`) — pegar o ano do período no cabeçalho.
- **Número que não é dinheiro**: `101.004` (nº de documento) parece valor BR. Exigir **vírgula decimal** (`\d,\d{2}`) resolve.
- **Rodapé colando na descrição**: no Nubank o rodapé ("Tem alguma dúvida?…") virava continuação da última transação da página. Transações são indentadas e o rodapé começa na coluna 1 — exigir indentação resolve.
- **PDF com senha**: o C6 exporta protegido. `pdftotext -upw SENHA` abre; a mensagem de erro precisa distinguir "falta senha" de "senha errada".

## Nunca reconheça o banco pelo nome do banco

Armadilha custosa: **a marca aparece como contraparte nas transações**. Um extrato Sicredi cita "SIFRA" (pagamento feito à Sifra) e um extrato Belluno cita "SICREDI" — inclusive no cabeçalho. Casar por `/sicredi/i` mandou o extrato do Belluno para o leitor errado, que aplicou modo saldo num extrato sem saldo corrente e produziu valores absurdos.

**Reconhecer por marcador de LAYOUT**: `"Associado" + "Cooperativa"` (Sicredi), `"Saldo no início do dia"` (Belluno), `"REL. DE EXTRATO PERIÓDICO"` (C6). Estrutura não vira contraparte.

## Extração do texto

`pdftotext -layout` (poppler-utils). O `-layout` é obrigatório: sem ele as colunas viram uma sopa e não dá para separar valor de descrição. Entrada e saída por stdin/stdout, sem arquivo temporário.

**No Node use `spawn`, não `execFile`**: `execFile` não aceita a opção `input` (isso é do `execFileSync`), então o processo fica esperando stdin para sempre e a requisição trava. Ver [[Armadilhas de child_process no Node]].

PDF digitalizado (imagem) não tem texto para extrair — detectável por não ter `/Font`. Aí só OCR, ou pedir o OFX.

## OFX, quando existe, é melhor

OFX 1.x é **SGML**, não XML: as tags de folha às vezes vêm fechadas (Nubank fecha) e às vezes não. Um parser que assuma XML bem formado quebra com metade dos bancos — extrair por regex tolerante a fechamento ausente cobre os dois.

Validação forte quando se tem os dois formatos do mesmo extrato: ler OFX e PDF e comparar. No Nubank de fev/2025 bateram exatamente — 34 transações, 3 entradas somando 1.875,02 e 31 saídas somando 2.060,11, iguais ao resumo declarado no próprio PDF.

## Conexões
- Usado por: [[Questor BI]] (seção Conciliação)
- Contas contábeis do banco: [[Contas bancárias e layout de contabilização no Questor]]
- Relacionado: [[Agregar antes de juntar em tabelas gigantes no Postgres]] (mesma ideia: deixar o dado se validar)
