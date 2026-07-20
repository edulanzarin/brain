---
tags: [tipo/atomica, camada/principio, infra]
criado: 2026-07-20
---

# Configuração vem do ambiente, não do código

> Tudo que muda entre a minha máquina e o servidor é variável de ambiente. O código
> é idêntico nos dois lugares.

## A regra

Vai pro ambiente: porta publicada, string de conexão, segredo, URL externa, chave de
API, nível de log. Fica no código: regra de negócio, layout, validação — tudo que deve
se comportar igual em todo lugar.

Três detalhes que fazem funcionar na prática:

- **Toda variável tem padrão de dev** (`${APP_PORT:-4011}`), pra clonar o repo e subir
  sem preencher formulário. Segredo é a exceção: sem padrão, e falha alto se faltar.
- **Ler a variável uma vez**, num módulo de config validado no boot, em vez de
  `process.env.X` espalhado. Se faltar algo, o app morre no start com mensagem clara,
  não três telas adiante.
- **`.env.example` versionado**, `.env` no gitignore. O exemplo é a documentação de
  quais variáveis existem.

## Por que

A mesma imagem tem que subir em dev, em teste e em produção sem rebuild. Se a porta ou
o host do banco estiverem no código, cada ambiente vira um build diferente — e aí
"funciona na minha máquina" deixa de ser piada e vira o estado normal do projeto.

O outro motivo é segredo: o que está no código está no git, e o que está no git está
no histórico pra sempre, mesmo depois de apagado.

## Conexões
- Aplicações: [[Porta interna é constante, porta externa é configuração]] · [[O nome do projeto governa o nome dos recursos]]
- Irmã: [[Ambiente de dev sobe igual ao de produção]]
- Mapa: [[Base]] · [[Infra]]
