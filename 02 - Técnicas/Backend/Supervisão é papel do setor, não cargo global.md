---
tags: [tipo/atomica, camada/padrao, seguranca, dev/backend]
criado: 2026-09-01
---

# Supervisão é papel do setor, não cargo global

> Num sistema dividido em setores (filas, departamentos, equipes), "quem manda"
> não é uma coluna no cadastro da pessoa: é uma marca na **associação** dela com
> cada setor. Quem supervisiona o Fiscal não supervisiona o Departamento Pessoal
> só por ter um cargo bonito, e quem precisa dos dois recebe a marca nos dois.

## O erro que evita

Cargo global parece mais simples e vaza por baixo. Um `papel = 'supervisor'` na
tabela de usuários responde "esta pessoa supervisiona?" quando a pergunta real é
sempre "esta pessoa supervisiona **isto aqui**?". Como a resposta global é sim, ela
passa a valer em todo setor — inclusive nos que a pessoa nem atende.

O sintoma costuma aparecer tarde e de lado: alguém do Fiscal transfere uma conversa
do Departamento Pessoal, ou o relatório do setor vizinho abre para quem não deveria
vê-lo. Ninguém escreveu isso; foi o cargo que se espalhou.

## A costura

Duas perguntas separadas, em duas tabelas:

| Pergunta | Onde mora |
|---|---|
| Quem configura o **sistema**? | `usuarios.papel` — global de verdade |
| Quem atende e quem manda em **cada setor**? | `usuario_setor (usuario_id, setor_id, supervisor)` |

```sql
ALTER TABLE usuario_departamentos
  ADD COLUMN supervisor boolean NOT NULL DEFAULT false;
```

E a consequência que prova que a separação está certa: **quem configura o sistema
não vê automaticamente o atendimento de todo setor.** Um gestor que mexe em módulos,
departamentos e integrações continua sem enxergar a fila de um departamento em que
não está lotado. Se isso soar errado, o remédio é lotá-lo — explícito, uma linha por
setor — e não devolver o poder ao cargo.

O caso que mais convence é o inverso: o atendente sem nenhum poder de configuração
que supervisiona o próprio setor. Ele existe em qualquer escritório real, e cargo
global não consegue descrevê-lo.

## Três níveis, não dois

A célula "pessoa × setor" tem três estados, e vale desenhá-la assim na interface:
**fora**, **atende**, **supervisiona**. Não são duas marcações independentes —
supervisionar já inclui atender —, então é escolha única, não caixa acumulável
([[Escolha única e múltipla não usam o mesmo controle]]).

## O que mais vale lembrar

- **Ler é mais aberto que escrever.** Colega do mesmo setor lê o atendimento do
  outro (é como se cobre uma ausência) e deixa nota interna, mas não responde por
  cima de quem já atende — senão duas pessoas atendem o mesmo cliente sem saber uma
  da outra. Só quem supervisiona tira a conversa das mãos de alguém.
- **A permissão vira flags derivadas, não repetidas.** `souDono`, `souMembro`,
  `podeResponder`, `podeGerenciar` saem de uma função só, e tanto a tela quanto a
  ação do servidor leem dali. Cada lado calculando o seu é como os dois divergem.
- **A prova chama as funções do sistema, não uma cópia.** Um teste que reescreve a
  consulta prova a cópia. As travessias que importam: não vê o setor vizinho na
  lista, não alcança pelo id digitado, lê o do colega sem responder, e — a mais
  esquecida — **quem não está em setor nenhum não vê nada**, que é a armadilha da
  lista vazia de [[Escopo de dado se clampa no servidor, num funil só]].

## Conexões
- Princípio: [[Permissão se valida no servidor, não na interface]]
- Irmã: [[Escopo de dado se clampa no servidor, num funil só]] · [[Permissão composta por papéis somados, não exceção por usuário]] · [[Isolamento entre clientes é política do banco, não filtro na query]]
- Visto em: [[Navehub]] · [[CRM Contábil]]
- Mapa: [[Backend]]
