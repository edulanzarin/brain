---
tags: [tipo/atomica, camada/principio, dev/backend]
criado: 2026-07-27
---

# A definição em dado dirige o comportamento, não um caso no código

> Quando o mesmo comportamento varia por um eixo conhecido, o eixo vira **dado**
> (linha de tabela, entrada de catálogo) que uma peça de código lê — não um
> `if`/arquivo por caso.

## A ideia

Toda vez que aparece "mais um tipo de X" e a resposta é copiar um bloco de código,
o sistema paga na próxima variação. A alternativa é descrever a variação como
**dado** e escrever **uma** peça que interpreta esse dado. Some-se um caso = some-se
uma linha, não um `if`. O que é estável fica no schema/na peça; o que varia mora no
dado.

Sinais de que é hora de virar dado:
- a diferença entre casos é só rótulo/opções/faixa, não lógica nova;
- adicionar um caso hoje exige editar código em vários lugares;
- quem devia poder mudar (o usuário, o RH) não é quem mexe no código.

O cuidado: o dado precisa de **um interpretador confiável** e de validação de
verdade no ponto de autoridade (servidor), senão dado ruim vira comportamento ruim.

## Onde já apareceu (dois casos, mesma lição)

- **Catálogo de módulos** do Nexo: `MODULOS`/`SECOES_*` são a fonte única que
  dirige launcher, sidebar e gate de permissão. Módulo novo é uma linha, não três
  lugares para editar e um para esquecer.
- **Formulário montado pelo usuário**: a definição em `formulario_campo` (tipo +
  `config` jsonb) dirige o renderer, o preview e a validação — a RH cria formulário
  sem tocar em código nem em migration. Ver [[Formulário montado pelo usuário — a
  definição no banco dirige renderer e validação]].

## Conexões
- Depende de: [[Configuração vem do ambiente, não do código]]
- Visto em: [[Navetech Hub]]
- Mapa: [[Base]]
