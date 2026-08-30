---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-30
---

# Instrumentar o gesto no funil, e um teste cobra a classe de cada verbo

> Quando a trilha de auditoria vira placar, ela mede **a instrumentação**, não o trabalho: gesto sem `registrarAuditoria` simplesmente não existe no painel, e ninguém nota. Duas costuras resolvem: o evento que precisa valer em toda tela nasce no **funil por onde todas já passam**, e um **teste varre o código atrás dos verbos gravados** cobrando classificação para cada um.

## O problema

Instrumentar tela a tela tem dois furos, e os dois são silenciosos.

O primeiro é de **cobertura**: a tela de amanhã nasce sem o registro, porque
lembrar de colar a linha é processo, não estrutura. O placar então mostra queda
de produtividade onde houve só uma tela nova.

O segundo é de **classificação**: o verbo `x.novo.gesto` é gravado, o painel
agrupa por classe, e o verbo sem classe cai no balde "Outros". Ninguém investiga
"Outros" — o número está lá, plausível, e errado.

## A solução

**Instrumentação no funil.** O gesto que vale para todas as telas se registra no
ponto único por onde todas passam. Numa dashboard com "aplicar no botão", esse
ponto é o `executar` do rascunho de filtros — um lugar, quatro módulos, e a
próxima tela já nasce contada:

```ts
const executar = useCallback(() => {
  auditarConsulta(pathname, rascunho);   // o módulo sai do próprio caminho
  atualizar(rascunho);
}, [atualizar, rascunho, pathname]);
```

Um clique = **um** registro, mesmo que a tela dispare seis consultas. O que se
conta é o gesto de quem pediu, não a requisição.

**Teste que varre o código.** O catálogo de classes é código; o conjunto de
verbos gravados também. Um teste lê os fontes, extrai os verbos e cobra
classificação — e cobra o inverso, que catálogo não vire lista de desejos:

```ts
const semClasse = verbosRegistradosNoCodigo()
  .filter((v) => classeDaAcao(moduloDe(v), v) === "outros");
expect(semClasse).toEqual([]);
```

## O que mais vale lembrar

- **Verbo DERIVADO escapa da varredura.** Se o servidor monta a ação de partes
  (`` `${modulo}.${tipo}` ``), não há literal no código para o regex achar — foi
  assim que a exportação ficou fora de todo painel sem ninguém perceber. Esses
  entram no teste à mão, numa asserção própria.
- **O balde "Outros" mostra o identificador cru**, não um rótulo bonito. É a
  única tela onde um nome técnico deve aparecer: ele é o alarme.
- Derivar o módulo do caminho (`pathname.split("/")[1]`) exige lista fechada do
  que a trilha aceita — módulo fora dela sai calado, não grava lixo.
- O beacon do cliente nunca manda a ação crua: manda `modulo` + um `tipo` de
  lista fechada, e o servidor deriva. Corpo com tipo forjado cai no default, não
  vira verbo novo na trilha.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Depende de: [[A trilha de auditoria já é o placar de atividade, não crie tabela de métrica à parte]]
- Irmã: [[Escopo de dado se clampa no servidor, num funil só]] — mesmo funil único, para o outro invariante (o escopo)
- Visto em: [[Navetech Hub]] — aba No Nexo das Produtividades
- Mapa: [[Backend]]
