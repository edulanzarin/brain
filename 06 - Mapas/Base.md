---
tags: [tipo/moc]
criado: 2026-07-20
---

# Base

Os **princípios**: regras que não dependem de framework, de linguagem nem de projeto.
É a camada mais estável e a primeira a consultar quando aparece um problema novo —
quase sempre já existe um princípio que cobre o caso, e o trabalho vira só escrever a
técnica nova embaixo dele.

Meta-regras: [[Camadas do conhecimento - princípio, padrão, aplicação]] ·
[[Conhecimento pertence à base, não ao projeto]]

## Design — como a tela se organiza

Independe de CSS, de Tailwind e de React. Técnicas concretas em [[Design]].

- [[Token semântico em vez de valor literal]] — nenhum valor cru no componente.
- [[Escala fechada em vez de valor solto]] — espaço, texto e raio saem de conjunto fixo.
- [[Container tem largura máxima e respiro constante]] — as três medidas de todo container.
- [[Hierarquia por superfície, não por borda]] — profundidade por fundo; borda é último recurso.
- [[Todo estado da tela tem visual]] — carregando, vazio, erro e sucesso são design.
- [[Estado compartilhável mora na URL]] — o que descreve a vista vai pro link.
- [[Estado de tela pertence à seção, não à página]] — estado no menor escopo que resolve.

## Infraestrutura — como um projeto sobe

Independe de Docker. Técnicas concretas em [[Infra]].

- [[Configuração vem do ambiente, não do código]] — a raiz de todas as outras.
- [[O nome do projeto governa o nome dos recursos]] — `prospects-app`, `prospects-db`.
- [[Porta interna é constante, porta externa é configuração]] — porta se escolhe na chamada.
- [[Uma faixa de portas por projeto]] — o mapa de portas reservadas.
- [[Ambiente de dev sobe igual ao de produção]] — mesma receita nos dois lugares.

## Segurança

- [[Permissão se valida no servidor, não na interface]] — esconder botão não é segurança.

## Ofício — como eu trabalho

- [[Verificar no build de produção, não só em dev]]
- [[Plataforma de IA hospedada prende o app pelo banco]]

## Cérebro — como este vault funciona

- [[Camadas do conhecimento - princípio, padrão, aplicação]]
- [[Conhecimento pertence à base, não ao projeto]]

## Como esta camada cresce

Princípio novo entra quando o **mesmo aprendizado aparece pela segunda vez**, em
contexto diferente. Antes disso é técnica, e técnica mora em [[Design]], [[Infra]],
[[Frontend]], [[Backend]] ou [[Dados]].

Princípio que nunca teve dois casos concretos costuma ser regra inventada, não regra
aprendida — e regra inventada é o que faz base virar entulho.

---

Voltar para [[Início]]
