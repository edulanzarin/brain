---
tags: [tipo/atomica, camada/padrao, dev/backend]
criado: 2026-08-24
---

# Rotação por período se apura na leitura, e dispensa agendador

> "Toda segunda troca o destaque", "todo dia zera o ranking", "a cada três dias
> entra o próximo": o reflexo é criar um cron. Quase sempre não precisa. Guarde
> **quando o período acaba** e apure na primeira leitura que acontecer depois
> disso.

## A forma

```sql
-- uma linha só, e a trava garante isso
CREATE TABLE reinado (
  id      integer PRIMARY KEY DEFAULT 1 CHECK (id = 1),
  item_id integer NOT NULL,
  fim     timestamptz NOT NULL
);
```

Na leitura: se `fim` ainda está no futuro, devolve o que está lá. Se já passou,
apura a janela que terminou, grava o próximo e devolve.

## Por que é melhor que o cron

- **Não há peça que falhe calada de madrugada.** Cron que não disparou não deixa
  sintoma até alguém olhar a tela e achar o valor velho.
- **Se auto-conserta.** Um dia inteiro sem visita nenhuma não deixa o sistema num
  estado inválido — só adia a troca até a próxima visita, que é quando ela
  importa.
- **Uma peça a menos pra existir**, e num serviço que talvez nem tenha agendador
  (contêiner sem cron, plataforma serverless).

## A corrida, que é o único jeito de errar isto

Duas leituras simultâneas no instante da virada apuram as duas. A gravação tem de
ser condicional ao período **ainda estar vencido**:

```sql
INSERT INTO reinado (id, item_id, fim) VALUES (1, $1, $2)
ON CONFLICT (id) DO UPDATE
   SET item_id = EXCLUDED.item_id, fim = EXCLUDED.fim
 WHERE reinado.fim <= now()   -- a segunda perde, e perde sem estragar nada
```

Sem o `WHERE`, a segunda visita estica o mandato do vencedor da primeira.

## Dois detalhes que se pagam

**A janela de apuração é o próprio período que acabou**, e não "os últimos N
dias". Numa apuração atrasada os dois recortes divergem, e o segundo premia uso
que já pertenceu a outro mandato.

**A poda mora aqui.** Apagar registro velho junto da apuração roda uma vez por
período, e não a cada leitura — então não precisa de rotina própria também.

## O teto também se conta pelo que está vivo

Quando o plano diz um número — "5 stories" —, a tentação é contar por data de
calendário: quantos entraram hoje. Isso reintroduz pela porta dos fundos o que a
expiração na leitura tinha resolvido, porque agora é preciso decidir que hora a
casa chama de meia-noite, e a resposta muda com o fuso do servidor.

Conte **quantos estão vivos agora**. Com prazo de 24 h, "no ar neste instante" e
"entraram no último dia" são a mesma pergunta, e a primeira sai da mesma consulta
que já filtra por `expiraEm > agora`. A oferta muda de redação junto: não é "5
por dia", é "5 no ar".

## Conexões
- Princípio: [[Estado mutável se lê da fonte no uso, não de cópia guardada]]
- Irmã: [[Contador de popularidade conta votante, não evento]]
- Visto em: [[piwdex2]]
- Mapa: [[Backend]]
