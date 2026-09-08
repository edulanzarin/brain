---
tags: [tipo/atomica, camada/padrao, dev/backend, dados]
criado: 2026-09-08
---

# Relatório de sistema logado sai do endpoint que a tela chama, não do DOM

> Pediram um relatório de um sistema web com login. O reflexo é automatizar o
> navegador e raspar a tabela. Mas a tabela é o que sobrou do dado: a tela já
> chamou um endpoint que devolveu tudo, com tipo e com as colunas que ela não
> desenha. Raspe o endpoint, não o desenho dele.

## O problema

Raspar a tela custa caro em três moedas, e as três só se pagam no fim:

1. **A tela é recorte.** No CRM de cancelamento da UFit, a aba "Lista" mostrava
   oito colunas — nome, CPF, telefone, unidade, etapa, SDR, atualizado, valor. O
   registro que alimenta essa linha tem **vinte e sete campos**, e os que
   faltavam eram exatamente os do relatório: saldo do contrato, multa retida,
   valor a devolver, valor estornado, data do estorno, meses usados. Nenhum
   deles aparece em pixel nenhum.
2. **A tela é formatação, e formatação é perda.** `R$ 1.234,56` é string, e
   virar número de volta depende de adivinhar a localidade. "há 6 dias" é pior:
   é uma data que foi apagada. O JSON entrega `1234.56` e
   `2026-03-25T12:00:48.137Z`.
3. **A tela é parcial por construção.** Lista virtualizada só monta no DOM o que
   cabe na viewport; paginação exige um laço de cliques que quebra quando alguém
   muda o rótulo do botão.

## A solução

O navegador é usado para **conseguir a sessão**, não para ler o dado. Três
passos:

```js
// 1. logar uma vez e guardar o cookie
await page.fill('#email', user); await page.fill('#password', pass);
await page.click('button:has-text("ENTRAR")');
await ctx.storageState({ path: 'state.json' });   // reusar nas próximas rodadas

// 2. escutar a rede enquanto a tela carrega, e ver quem traz o volume
page.on('response', async (r) => {
  const ct = r.headers()['content-type'] || '';
  if (ct.includes('json')) console.log(r.status(), r.url(), (await r.body()).length);
});
// -> 200 /api/crm/leads?cargoId=...&limit=2000   1.419.187 bytes   <- é esse

// 3. repetir a chamada DE DENTRO da página
const dados = await page.evaluate(async (url) => {
  const r = await fetch(url, { credentials: 'include' });
  return r.text();
}, '/api/crm/leads?cargoId=...&limit=5000');
```

O passo 3 é o que economiza a tarde. Chamando de dentro do documento, **cookie,
origem, cabeçalho de CSRF e qualquer token que o cliente injeta vão junto de
graça** — não é preciso reconstruir a autenticação em `curl` nem descobrir como
o app assina a requisição. É a mesma sessão, o mesmo código, só sem a tela.

O tamanho da resposta é o que denuncia o endpoint certo: entre uma dúzia de
chamadas de sessão e permissão com 40 a 300 bytes, a que traz o relatório
aparece sozinha com um megabyte.

## O que mais vale lembrar

- **Suba o `limit` e confira o total.** A tela pediu `limit=2000` e o payload
  voltou com `{leads, total, page, limit}`: `total` igual ao tamanho do array é
  a prova de que veio tudo numa chamada. Sem esse par, você não sabe se está
  olhando a primeira página.
- **GET desconhecido é seguro de sondar; escrita não.** Ler é read-only, e o
  sistema é produção de outra pessoa. A mesma disciplina de
  [[O bundle público do cliente entrega o contrato da API sem documentação]].
- **Guarde o `storageState`.** Cada iteração do relatório vira uma chamada, não
  um login. É o que torna o trabalho repetível sem virar carga no servidor
  alheio — e sem subir navegador toda hora.
- **Ainda vale abrir a tela.** Ela nomeia o que o JSON numera: foi a coluna do
  Kanban que explicou que `stageId` é etapa do funil e em que ordem elas vêm.
  Ver [[A interface do sistema explica o que a API dele esconde]].
- **O que a tela mostra e o payload não tem, o payload não deve.** Se um número
  aparece na tela e não está no JSON, ele é calculado no cliente — e aí a conta,
  não o campo, é o que você precisa copiar.

## Conexões
- Princípio: [[Auditar o registro, não só o agregado]] — a tela é a lente
  agregada (oito colunas curadas); o relatório precisa do registro inteiro.
- Irmã: [[O bundle público do cliente entrega o contrato da API sem documentação]] ·
  [[A interface do sistema explica o que a API dele esconde]] ·
  [[Quando a REST não expõe o dado, o WebSocket do mesmo sistema entrega]] ·
  [[Importação em massa passa pela API, não pelo banco]]
- Visto em: relatório de cancelamentos da UFit Academia (set/2026)
- Mapa: [[Backend]]
