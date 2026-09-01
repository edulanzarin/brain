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
- **Teto real ~45 req/min**, não os 100 da doc. Calibrado com 20 chamadas por
  taxa em ago/2026: a 80/min, 12 de 20 voltaram 429; a 60/min, 10 de 20; a
  45/min, zero. Não há header `Retry-After` nem `X-RateLimit-*` — nada diz
  quanto esperar.
- **A latência domina o ritmo.** Cada chamada de `deliveries` leva ~1,4s
  independentemente do tamanho da janela de datas (medido com 3, 8 e 18 meses),
  o que é MAIOR que o intervalo de 45/min. Na prática o espaçamento vira
  irrelevante e a varredura anda no passo da rede. **Medido ponta a ponta em
  ago/2026: 1.575 empresas ativas em 46 min, 7.331 entregas pendentes, zero
  falhas.**
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
- **`situation` inválido é IGNORADO em silêncio.** Medido: `situation=finalizada`
  (valor que não existe) devolveu **byte a byte o mesmo** que sem filtro nenhum —
  25.818 bytes nos dois. Um erro de grafia no filtro não dá erro: devolve tudo, e
  quem chamou acha que filtrou. Mesma família da armadilha do `?config=1`, e mais
  perigosa, porque o resultado *parece* certo. Dos três valores documentados,
  `delivered` funciona e `pending`/`read` devolvem **204** quando não há nada
  naquela situação (não uma lista vazia).
- **`ListAll` não vale para tudo — mas em `deliveries` vale, e a medição
  anterior estava errada.** `deliveries/ListAll` devolve 204 *sozinho*; com
  **`DtLastDH`** junto, devolve 200 e a carteira inteira (ver a seção da
  varredura incremental abaixo). Foi medido sem o parâmetro e concluído
  "não funciona" — a doc dizia, e a doc estava certa desta vez. `invoices`
  segue devolvendo 204 em tudo que se testou. A barra do CNPJ vai crua na URL —
  encodar quebra.
- **Parâmetro de presença não aceita valor.** `?config` traz o bloco Config em
  cada entrega; `?config=1` devolve **HTTP 204, corpo vazio**. Como todo
  construtor de query escreve `chave=valor`, é fácil implementar a forma errada
  depois de testar a certa na mão — ver
  [[Parâmetro de presença perde o efeito se você der um valor a ele]].
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

## Varredura incremental global (medido set/2026)

`GET /deliveries/ListAll?DtInitial=…&DtFinal=…&DtLastDH=…&config` devolve **a
carteira inteira** numa varredura paginada, em vez de uma chamada por CNPJ.

- **Custo medido**: 56 páginas, 648 empresas, **2.800 entregas em 58,7 s** para
  UM dia de mudanças. A varredura por CNPJ que faz o mesmo trabalho leva **46
  minutos** (1.575 empresas). É ~47× mais barato manter fresco.
- **A página é de 50 ENTREGAS, não de 50 empresas.** O número de empresas por
  página variou de 2 a 34 enquanto toda página trazia exatamente 50 entregas.
  Quem parar contando empresas trunca a varredura no primeiro lote pequeno.
- **`DtLastDH` só aceita o dia atual ou o anterior**, como a doc diz. Data mais
  velha devolve **204 com corpo vazio** — e esse é o mesmo 204 de "nada mudou
  desde então". Os dois casos são indistinguíveis pela resposta, então quem
  passar uma data velha conclui "está tudo em dia" e não vê erro nenhum.
  Medido: `2026-08-31` → 200 com 18 KB; `2026-08-25`, `2026-07-01` e
  `2026-01-01` → 204, todos.

Ou seja: **ListAll é o delta, não o retrato**. A carga inicial (ou qualquer
recuperação de janela perdida) continua sendo empresa a empresa; o dia a dia é
uma varredura de um minuto.

## O histórico de entrega existe, com autor e carimbo

`situation=delivered` devolve o que foi entregue, e o corpo é completo. Medido
numa empresa (14 entregas de jun a ago/2026), **100% de preenchimento** em:

| campo | o que é |
|---|---|
| `EntDtEntrega` | data da entrega |
| `EntDtFinalizacao` | **timestamp** da finalização (`2026-07-01 08:52:45`) |
| `RespEntrega` / `Config.RespEntregaID` | **quem entregou** (nome e id de usuário) |
| `Config.RespPrazo` / `RespPrazoID` | quem era o responsável pelo prazo |
| `Config.DptoNome` | o setor (Contábil, Fiscal, Pessoal…) |
| `EntCompetencia`, `EntDtPrazo`, `EntDtAtraso`, `EntMulta` | a régua para medir atraso |

`RespPrazo` e `RespEntrega` **podem divergir** — quem devia e quem fez. A
diferença é, ela mesma, um indicador.

Com isso o Acessórias deixa de ser só uma fila de pendências e vira fonte de
**produção entregue por pessoa**, nos três setores.

## Os recursos que ninguém usou ainda (medido set/2026)

| recurso | volume | o que tem de útil |
|---|---|---|
| `requests/ListAll` | **865 solicitações em ago/2026** (45 páginas, ~8 s) | `SolDHAbertura`, `SolDHFinalizacao`, **`SolUsuarioFinalizador`+ID**, `SolOfficeResp[]`, `DptoID/Nome`, prazo, prioridade, status (`Finalizada`/`Nova`/`Resolvendo`). 61% finalizadas com carimbo, por 34 pessoas |
| `processes/ListAll` | ≥800 em 2026 | `ProcGestor`, `ProcInicio`, `ProcConclusao`, `ProcPorcentagem`, `ProcDepartamento`, `ProcStatus`. 54 pessoas |
| `company_groups/ListAll` | **435 grupos** | id, nome, status. Cadastro de grupo de empresas que já existe lá — o Nexo mantém o dele à parte |
| `users/ListAll` | pagina de 20 | id → nome → e-mail → status. É o que resolve `RespEntregaID` em gente, e diz quem já saiu |
| `tags/ListAll` | 20+ | marcadores de empresa |

Armadilha do grupo: `company_groups/ListAll&companies=1` devolve **erro dentro
de um 200** ("Para listar empresas, informe o ID do grupo"). A flag só vale com
um id no caminho; para listar os grupos, vai sem ela.

## Página curta NÃO significa fim de lista — e depende do endpoint

Medição que muda como se escreve o laço:

- **`/requests/ListAll`**: há **uma página curta no MEIO** da lista, de forma
  reprodutível (duas rodadas idênticas, 865 itens em 45 páginas, a curta sempre
  no mesmo lugar). Parar em `lote.length < 20` trunca em ~metade.
- **`/deliveries/{cnpj}`**: as páginas vêm 50, 50, …, 46 — só a última é curta.
  Ali o critério por tamanho é seguro, e é o que o [[Navetech Hub]] usa.

Não dá para generalizar o critério de parada entre endpoints da mesma API.

### O que torna a varredura determinística é o ESPAÇAMENTO, não a contagem de vazios

A mesma consulta, três rodadas **sem espaçar as chamadas**: 865, 660 e **0**
itens. Com **1,4 s entre chamadas**: 865 nas duas rodadas, idêntico.

Isso refina a regra do vazio-como-suspeita: repetir a chamada não adianta se as
repetições caem todas dentro da mesma janela de throttle. O que salva é o
intervalo entre elas — e, na dúvida, recuo crescente antes de repetir. Uma
varredura "rápida" desta API não é rápida: é uma que mente.

## Casar com o Questor

O `Identificador` do Acessórias é o CNPJ formatado, mesmo formato de
`estab.inscrfederal` no Questor. Casar por **qualquer estabelecimento**, não só a
matriz (`codigoestab = 1`): o Acessórias cadastra **filial como empresa
própria**. Medido em 1.200 ativas — só matriz casa 75%; com todos os estabs,
90%. Os ~10% restantes são clientes reais sem par no Questor, e o que fazer com
eles é [[Dado externo sem par no cadastro local não tem escopo]].

## Conexões
- Depende de: [[Quando a API cobra uma chamada por item, filtrar não economiza]]
- Como varrer sem truncar: [[Fim de lista se prova com intervalo, não com repetição]]
- Irmã: [[Dado externo sem par no cadastro local não tem escopo]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Backend]]
