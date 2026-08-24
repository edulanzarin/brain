---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-24
---

# Contador de popularidade conta votante, não evento

> Qualquer coisa que se anuncie como "o mais visto", "em alta" ou "mais buscado"
> e conte **eventos** está medindo quem recarrega mais, não quem interessa mais.
> É um placar de F5. A unidade tem de ser a pessoa, e uma vez por período.

## A forma

Chave primária composta faz o trabalho, sem código de deduplicação:

```sql
CREATE TABLE uso (
  item_id integer NOT NULL,
  dia     date    NOT NULL,
  votante text    NOT NULL,
  PRIMARY KEY (item_id, dia, votante)
);
-- INSERT ... ON CONFLICT DO NOTHING
```

O spam vira `DO NOTHING`: não há caminho no código onde ele conte. Medido no caso
real, 20 pedidos do mesmo votante viraram 1 voto — e não porque alguém escreveu
um `if`.

## O `votante` é hash com sal e DATA, e isso não é detalhe

```
votante = sha256(segredo | dia | ip | user-agent)
```

Três propriedades, e as três importam:

- **o IP não é guardado** — só o resultado, do qual não se volta;
- **a data entra no hash**, então o código muda sozinho toda meia-noite: serve pra
  não contar repetido dentro do dia e não serve pra ligar duas visitas em dias
  diferentes;
- **o segredo entra**, então o mesmo IP em dois sistemas não gera o mesmo código.

Ou seja: é deduplicação, e não identificação. A diferença é o que separa um
contador de um rastreador, e ela mora inteira nessas três linhas.

## A página de privacidade é parte da feature

Se o site diz "não guardo nada sobre você", a primeira feature que guarda
transforma essa frase em mentira — e ninguém vai lembrar de conferir. Texto de
privacidade que afirma ausência é um **acoplamento**: quem adiciona persistência
tem de abrir aquele arquivo no mesmo commit.

## O que não escolher

Cookie ou `localStorage` como identidade de voto parece mais limpo e é pior: é
estado no cliente, some na aba anônima, e (ao contrário do hash diário) tende a
ser *permanente*, que é exatamente o que não se queria.

## Conexões
- Princípio: [[A tela não afirma mais precisão do que a fonte tem]]
- Irmã: [[Rotação por período se apura na leitura, e dispensa agendador]] ·
  [[Peça o que a fonte mostra, não o que você precisa]]
- Visto em: [[piwdex2]]
- Mapa: [[Backend]]
