---
tags: [tipo/atomica, camada/principio, dev/backend]
criado: 2026-08-17
---

# Guarde a intenção e o processo se reconstrói dela

> O que o usuário QUER se persiste separado do que está acontecendo; qualquer processo que morra se reconstrói lendo a intenção.

## A regra

Estado vivo (a conexão aberta, o timer rodando, o job em memória) morre com o
processo — restart, deploy, queda. Se a única memória do sistema é o estado vivo, a
morte é silenciosa: nada indica que aquilo devia estar rodando. A saída é separar
**intenção** de **execução**: a intenção ("o robô deve estar caçando em X, no modo Y")
vive no banco; a execução é derivada — no boot e na reconexão, o processo lê a
intenção e se reconstrói.

Teste rápido: derrube o processo. Se ao subir ele não volta a fazer o que o usuário
pediu, a intenção estava presa na memória — errado.

## Por que

Intenção e execução mudam em momentos diferentes e têm donos diferentes: a intenção
muda quando o usuário decide; a execução muda quando o mundo interfere (queda de
rede, restart). Misturá-las faz a interferência do mundo apagar a decisão do usuário.
Separadas, a decisão sobrevive a qualquer interferência — e a reconexão automática
deixa de ser um caso especial: é só o processo re-derivando a execução da intenção.

Casos concretos: o robô do piwdex guardava tudo em memória — restart do container
matava a caçada sem aviso; a tabela de estado desejado religou tudo sozinho
([[Estado desejado persistido religa o robô depois do restart]]). No CRM, a regra de
envio recorrente é a intenção persistida que materializa campanhas e se reprograma
([[Regra de envio recorrente materializa uma campanha e reprograma]]).

## Conexões
- Técnica: [[Estado desejado persistido religa o robô depois do restart]]
- Técnica: [[Regra de envio recorrente materializa uma campanha e reprograma]]
- Mapa: [[Base]]
