---
tags: [tipo/atomica, camada/referencia, dev/backend]
criado: 2026-08-28
---

# API do Acessórias

Referência da API REST do **Acessórias**, o sistema onde a Navecon controla as
obrigações entregues ao cliente. Doc oficial em `api.acessorias.com/documentation`
— o que está aqui é o que foi **medido contra a API real**, incluindo onde ela
diverge da doc.

## Acesso

- **Bearer token** no header. Gerado no próprio sistema: engrenagem > "API Token".
- O token é de **escritório**, não de empresa — alcança a carteira inteira — mas
  herda as permissões do USUÁRIO que o criou. Integração séria pede um usuário
  administrativo dedicado; token preso a conta pessoal cai quando a pessoa sai.
- **Teto de 100 req/min**, sliding window, HTTP 429 ao estourar.
- **Não há webhook.** Nada avisa mudança; quem quer o dado fresco varre de novo.
  Existe sincronização incremental por `DtLastDH` em `companies` e `deliveries`.

## Divergências da doc que custam caro

- **Erro com HTTP 200.** Parte das falhas volta 200 com a chave `Erro` no corpo
  (a própria doc admite, como legado). Quem checa só o status engole falha como
  sucesso — a checagem tem que ser do CORPO.
- **O vocabulário de status é maior que o documentado.** A doc promete
  `pending/read/delivered`; a API devolve nove rótulos: `Pendente`, `Atrasada!`,
  `Dispensada`, `Ent. atrasada`, `Ent. antecipada`, `Ent. PzTéc`, `Prazo técnico`,
  `Atraso justificado`, `Ent. justificada`. Guardar como vem é mais honesto que
  inventar um de-para.
- **`ListAll` não vale para tudo.** `deliveries` e `invoices` devolvem 204 com
  ele; `deliveries` exige o identificador (CNPJ/CPF) no caminho, e o id numérico
  da empresa também não serve. A barra do CNPJ vai crua na URL — encodar quebra.
- **Filtro por setor só existe em `deliveries`.** Em `requests` e `processes` o
  parâmetro é aceito e **ignorado em silêncio** (testadas quatro grafias): volta
  o conjunto misturado, sem erro. O setor vem no corpo; o recorte é no cliente.

## O que cada recurso dá

| Recurso | Setor no corpo | Filtra por setor | Observação |
|---|---|---|---|
| `deliveries/{cnpj}` | sim, com a flag `config` | **sim**, `department_id` (lista por vírgula) | exige `DtInitial`/`DtFinal`; pagina de 50 |
| `companies` | `?departments` traz setores **+ responsável** de cada um | sim (`?departments=1,3,5`) | pagina de 20; `DtLastDH` incremental |
| `requests` | `DptoID`/`DptoNome` | não | pagina de 20 |
| `processes` | `ProcDepartamento` (nome, sem id) | não | pagina de 20 |
| `users` | **nenhum** | — | só id, nome, e-mail, status |
| `matrices` | campo `departamento` quase sempre vazio | — | pouco aproveitável |
| `invoices` | — | — | 204 em tudo que se testou |

`deliveries` com `config` é o recurso mais rico da API: por entrega traz setor,
`RespPrazo`/`RespPrazoID`, `RespEntrega`, competência, prazo, multa e o `EntID`
(chave estável, boa para materializar sem duplicar).

**`situation=pending`** devolve exatamente o acionável (`Atrasada!` + `Pendente`)
e corta o payload em ~9x — numa empresa medida, 2 linhas contra 18.

## Casar com o Questor

O `Identificador` do Acessórias é o CNPJ formatado, mesmo formato de
`estab.inscrfederal` no Questor. Casar por **qualquer estabelecimento**, não só a
matriz (`codigoestab = 1`): o Acessórias cadastra **filial como empresa
própria**. Medido em 1.200 ativas — só matriz casa 75%; com todos os estabs,
90%. Os ~10% restantes são clientes reais sem par no Questor, e o que fazer com
eles é [[Dado externo sem par no cadastro local não tem escopo]].

## Conexões
- Depende de: [[Quando a API cobra uma chamada por item, filtrar não economiza]]
- Irmã: [[Dado externo sem par no cadastro local não tem escopo]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Backend]]
