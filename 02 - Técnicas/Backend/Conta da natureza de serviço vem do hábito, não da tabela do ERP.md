---
tags: [tipo/atomica, camada/padrao, dev/backend, sql, armadilha]
criado: 2026-08-26
---

# Conta da natureza de serviço vem do hábito, não da tabela do ERP

> A natureza de serviço do ERP tem tabela de contabilização como qualquer CFOP —
> só que ela envelhece. Quem manda na conta esperada é a conta que a natureza de
> fato recebe nos últimos 12 meses; sem uma conta dominante, não há regra e a
> nota espelha.

## O problema

Reproduzir a contabilização de uma nota é ler a tabela do ERP: natureza → tabela
de contabilização → conta + fórmula. Para mercadoria isso é regra viva, porque a
tabela é o que o próprio ERP executa. Para **serviço**, a mesma leitura produz
erro em massa, por dois motivos diferentes:

1. **Config morta.** A empresa cadastra uma conta nova — às vezes com o mesmo
   nome e o mesmo apelido da antiga — e repõe o lançamento nela, mas a natureza
   continua apontando pra velha. Medido no Questor (mai–jul/2026, 11.866 NFSE de
   natureza única): a conta da tabela acerta **62%**; a conta habitual da
   natureza acerta **86%**. Em 443 pares (empresa, filial, natureza) TODAS as
   notas caem numa conta só, diferente da configurada — e em **79%** deles a
   conta configurada não teve nenhum movimento no trimestre.
2. **Natureza genérica.** "Serviços Tomados S/ Retenção – Serv. Profiss." recebe
   de tudo, e a conta muda de nota para nota **de propósito**. Ali nenhuma conta
   domina: 3.424 notas viviam nesse caso, e 2.612 delas eram acusadas de conta
   errada — em natureza genérica, "diferente da tabela" é o normal.

O sintoma dos dois é o mesmo e é o pior possível: dezenas de notas marcadas por
empresa, todo mês, sempre as mesmas. A tela vira ruído e para de ser lida.

## A solução

Aprender a conta por (empresa, filial, natureza) e guardar com a evidência:

```sql
-- moda da conta do lançamento PRINCIPAL, 12 meses, só nota de UMA natureza
-- (em nota multi-natureza não dá pra atribuir a conta a uma delas)
with uma as (
  select chave, min(estab) estab, min(cfop) cfop
    from prod group by chave having count(*) = 1
), principal as (            -- entrada debita despesa, saída credita receita:
  select distinct on (chave) -- o principal é o maior valor daquele lado
         chave, conta from real order by chave, v desc
)
select u.estab, u.cfop, p.conta, count(*) n
  from uma u join principal p on p.chave = u.chave group by 1,2,3
```

Depois, três decisões e não uma:

- **≥ 3 notas e a moda ≥ 80%** → é regra: o motor espera a conta habitual, não a
  configurada. Ver [[Config declarada envelhece; quem diz a regra é o comportamento observado]].
- **sem dominância** → não há regra: a linha principal do plano passa a ser
  tratada como **conta variável** (a mesma marca que o ERP usa para a
  contrapartida do fornecedor, cuja conta só existe no lançamento). Um flag só e
  os dois consumidores — conferência e balancete — já se comportam certo, sem
  conceito novo.
- **menos de 3 notas** → o histórico não diz nada; fica valendo o ERP.

## O que mais vale lembrar

- **O espelho precisa ser por NOTA, não por conta.** A nota sem regra costuma
  cair numa conta que OUTRAS naturezas regram; um gate por conta recusa o
  espelho e a nota fica sem contrapartida no lado esperado — vira uma falta do
  tamanho exato dela. [[Onde não há regra, espelhar é mais honesto que arbitrar]].
- **Só o componente principal.** As linhas de tributo (PIS/COFINS a recuperar)
  seguem a tabela do ERP, que nelas não envelhece — trocá-las junto criaria erro
  onde não havia.
- **A tela de configuração continua mostrando o plano cru.** O aprendizado é do
  conferidor; quem for consertar o cadastro no ERP precisa ver o que o ERP diz.
- Nada disso vale para mercadoria: lá a tabela é executada pelo próprio ERP, e
  aprender por cima esconderia erro de verdade.

## Conexões
- Princípio: [[Config declarada envelhece; quem diz a regra é o comportamento observado]]
- Irmã: [[Onde não há regra, espelhar é mais honesto que arbitrar]]
- Parente: [[Número de regra alheia se lê da fonte, não se congela em constante]] · [[O cálculo puro sai do módulo server-only para poder ser testado]]
- Referência do banco: [[NFSE não tem regra de conta, o fiscal decide na hora]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Backend]]
