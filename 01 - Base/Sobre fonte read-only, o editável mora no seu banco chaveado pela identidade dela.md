---
tags: [tipo/atomica, camada/principio, dev/backend, sql]
criado: 2026-08-12
---

# Sobre fonte read-only, o editável mora no seu banco chaveado pela identidade dela

> A fonte que você não pode escrever é espelho; o que você edita vive do seu lado, referenciado pela identidade dela.

## A regra

Quando a fonte de dados é imutável para você — produção de terceiro, ERP externo,
banco só-leitura — a camada editável não vive nela. Vive no **seu** banco, chaveada
pela **identidade natural da fonte** (as chaves dela como valores, sem FK cruzando
bancos), e o valor efetivo é um merge na aplicação: `override ?? fonte`.

Isso cobre os três tipos de edição:
- **corrigir** um registro que veio errado → tabela de override com só os campos sobrescritos;
- **incluir** uma entidade que não existe na fonte → tabela local com id próprio (e um id sintético quando ela precisa se passar por um registro da fonte);
- **renomear/reclassificar** → tabela que sobrepõe o rótulo por identidade.

## Por que

Escrever na fonte é impossível ou proibido, e copiar a fonte inteira para poder
editá-la cria uma segunda verdade que desatualiza. Guardar só o **delta**, ancorado
na identidade da fonte, mantém a fonte como verdade única e ainda te deixa editar sem
tocá-la. Quando a fonte muda, o delta continua válido porque é a identidade que os une,
não uma cópia.

Caso concreto (Nexo): o Questor é produção só-leitura e vinha bagunçado. A RH precisava
corrigir setor/cargo de funcionários, incluir prestadores PJ que não existem lá e
renomear setores. Tudo virou camada gravável no banco do app, chaveada por
`(codigoempresa, codigofunccontr, classiforgan)`. O Questor não foi tocado.

## Na prática

O merge é seu, feito no servidor — nunca um JOIN cruzando os dois bancos. A mesma
forma já tinha aparecido antes de virar regra: uma worklist que recomputa o achado da
fonte e persiste só a triagem humana, e um de-para que casa dois sistemas e salva só o
override do humano. Três casos, mesma espinha: o editável do seu lado, a fonte intacta.

## Conexões
- Irmã: [[Migração de dados mantém o antigo como reserva até a virada]]
- Técnica que aplica: [[O que o Questor não dá mora no app-db chaveado pela identidade dele]]
- Mapa: [[Base]]
