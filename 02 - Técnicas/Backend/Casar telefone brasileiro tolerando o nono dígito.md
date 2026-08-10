---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-08-10
---

# Casar telefone brasileiro tolerando o nono dígito

> No Brasil o mesmo celular tem **duas grafias**: com e sem o **9** depois do DDD. O
> WhatsApp (`wa_id`) costuma devolver **sem** o 9 (12 dígitos: `55` + DDD + 8), enquanto o
> cadastro guarda **com** o 9 (13 dígitos). Casar por igualdade literal **duplica o contato**.
> Case por **classe de equivalência**: gere as duas variantes e busque nas duas.

## O gerador de variantes

Sempre em dígitos puros (sem `+`, sem máscara). Só mexe em número BR (`55`):

```
55 + DDD(2) + 8 dígitos   → insere o 9:  55 DDD 9 XXXXXXXX
55 + DDD(2) + 9 + 8 díg.  → remove o 9:  55 DDD  XXXXXXXX
```

Buscar `where phone IN [original, variante]`. Achou → é o mesmo contato. Não achou → cria
(guardando a grafia que chegou). Assim uma mensagem do `555491234567` encontra o contato
salvo como `5554991234567`, e não nasce um segundo.

## Onde morde

- **Ingestão de webhook**: mensagem recebida cria contato/conversa; sem o casamento, cada
  cliente vira dois registros e a conversa se parte.
- **Envio**: mandar pro número "errado" (com/sem 9) às vezes a operadora tolera, às vezes
  não. Padronizar na entrada evita o problema nos dois lados.

Vale pra qualquer sistema que troque telefone com o mundo real no Brasil — mensageria,
telefonia, cadastro que casa com base externa.

## Conexões
- Visto em: [[navetalks]]
- Mapa: [[Backend]]

<!-- Folha: o princípio-mãe ("casar dado do mundo real por classe de equivalência, não por
     igualdade literal") ainda não está na Base. Candidato quando aparecer um segundo caso
     (ex.: normalizar CNPJ com/sem máscara, ou nome com/sem acento). Não inventar agora. -->
