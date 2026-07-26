---
tags: [tipo/moc]
criado: 2026-07-20
---

# Backend

Técnicas de servidor: Node, rotas, arquivos, integrações. Princípios em [[Base]];
o que é banco de dados tem mapa próprio em [[Dados]].

## Processos e arquivos

- [[Armadilhas de child_process no Node]] — timeout, stderr e `EPIPE`; por que
  `spawn` em vez de `exec`.
- [[Ler extrato bancário em PDF]] — extrair dado estruturado de PDF.
- [[Servir anexo por rota com checagem de permissão]] — arquivo protegido não é
  arquivo estático. Princípio: [[Permissão se valida no servidor, não na interface]].
- [[Trocar o backend de armazenamento sem downtime]] — mover binário do banco pra
  uma pasta com ponteiro, leitura de reserva e migração sob demanda. Princípio:
  [[Migração de dados mantém o antigo como reserva até a virada]].

## Autenticação e permissão

- [[Cravar o seam de permissão antes do login]] — stub da sessão + gate num lugar só,
  pra permissão não virar retrofit. Princípio:
  [[Permissão se valida no servidor, não na interface]].
- [[Sessão opaca no banco separa autenticação de permissão]] — cookie só carrega o
  token; permissão vem do banco a cada request (revogável, sempre atual).
- [[Escopo de dado se clampa no servidor, num funil só]] — a lista de "o que o
  usuário pediu" não é confiável; a restrição real mora num funil único.
- [[O que dois módulos compartilham é a query, não a rota]] — reuso de dado entre
  módulos sem furar o gate por módulo.
- [[Formulário público por token opaco fica fora do gate de sessão]] — quem não
  tem login responde por link; o token é a credencial, a exceção ao gate é
  cirúrgica.
- [[Filtro transversal só é honesto se todo o funil o honra]] — dimensão de
  filtro nova (filial) precisa chegar a toda consulta; funil compartilhado
  propaga, query própria é buraco silencioso.

## Integrações

- [[Polling substitui webhook quando não há IP público]] — quando não dá pra receber
  chamada de fora. Princípio: [[Configuração vem do ambiente, não do código]].

## Princípios que mandam aqui

- [[Permissão se valida no servidor, não na interface]]
- [[Configuração vem do ambiente, não do código]]
- [[Ambiente de dev sobe igual ao de produção]]

---

Voltar para [[Início]]
