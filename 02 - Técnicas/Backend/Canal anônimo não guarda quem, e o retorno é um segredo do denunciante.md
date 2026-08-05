---
tags: [tipo/atomica, camada/padrao, seguranca, dev/backend]
criado: 2026-08-04
---

# Canal anônimo não guarda quem, e o retorno é um segredo do denunciante

> Quando o canal PROMETE anonimato (denúncia de assédio, pesquisa de clima), o
> anonimato não é uma política — é a **ausência da coluna**. Não se grava IP,
> user-agent, cookie nem vínculo com `usuario`/`sessao`. Se o dado que
> identificaria não existe no schema, nenhum vazamento, log ou subpoena o
> reconstrói. É [[Um invariante se garante na estrutura, não no processo]] virado
> pra privacidade: garanta na estrutura, não na promessa de não olhar.

## O aperto real

Anonimato briga com duas necessidades:

1. **O denunciante precisa acompanhar o caso** (a Lei 14.457/2022 espera retorno)
   — mas login o identificaria.
2. **A gestão precisa recortar o dado** (eNPS por setor) — mas um recorte fino
   deanonimiza um time pequeno.

A saída é resolver cada uma sem reintroduzir identidade:

- **Retorno via segredo que só o denunciante tem.** Ao enviar, gera-se um par
  `protocolo + senha` e mostra-se **uma vez**; a senha é guardada só como **hash**
  (o mesmo `scrypt` do login). Com o par, a pessoa volta, vê o status e troca
  mensagens — sem nunca dizer quem é. Perdido, é **irrecuperável**, e é assim que
  tem que ser: recuperável implicaria um canal de identidade que anularia o
  anonimato. Protocolo/senha inválidos caem no **mesmo 404**, pra não revelar se
  um protocolo existe (evita enumeração).
- **Supressão no agregado.** O recorte por setor (ou qualquer corte) só aparece
  acima de um **N mínimo** de respostas — abaixo disso, some. Anonimato não é só
  não guardar identidade na ENTRADA; é não deixar a SAÍDA reidentificar. Um único
  respondente de um setor com nota péssima é um dedo apontado.

## O que não fazer

- Guardar IP "só pra auditoria/antifraude": é o vínculo que a promessa disse não
  existir. Se precisa de antifraude, use um soft-guard client-side (localStorage)
  ciente de que link aberto não impede de verdade — não uma coluna que trai.
- Deixar a gestão ver o recorte cru "porque é interno": o interno é justamente
  quem tem meio de retaliar.

O canal em si segue [[Formulário público por token opaco fica fora do gate de
sessão]] (aberto, sem login, fora do gate) — a diferença é que aqui, além de não
exigir login, o sistema **deliberadamente não sabe** e não pode saber quem falou.

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[Formulário público por token opaco fica fora do gate de sessão]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Backend]]
