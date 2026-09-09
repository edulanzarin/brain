---
tags: [tipo/atomica, camada/principio, seguranca]
criado: 2026-09-09
---

# O recorte é a área que responde, não o assunto que a tela trata

> Um assunto que várias áreas tratam se instala DENTRO de cada área. Juntá-lo
> num lugar só parece organização, e o que ele faz é obrigar o sistema a inventar
> um leitor de tudo — um papel que a organização não tem.

## A regra

Quando a mesma tela serve várias áreas, o recorte que sobrevive é o da **área
que responde** por aquele trabalho, não o do **assunto** que a tela trata.

O assunto é um bom nome de pasta e um péssimo dono. Ele agrupa por semelhança,
e semelhança não é responsabilidade: quatro áreas preenchendo o mesmo formulário
continuam sendo quatro áreas, cada uma com o seu chefe, o seu processo e a sua
cobrança.

## Por que

O custo aparece na permissão, e aparece invertido — não como acesso negado, mas
como um cargo que passa a existir só porque o software precisou dele.

**O caso que ensinou.** Num sistema de escritório contábil, o relatório de
incidente (post mortem) passou a ser preenchido por quatro setores. Virou módulo
próprio: uma seção por setor, mais uma Visão geral que lia tudo. Desenho limpo
no papel. Na prática, a única leitura completa era a dessa Visão geral — quer
dizer, de uma coordenação central —, e **o gestor de cada setor ficou sem a
leitura da sua área**. Justamente quem cobra o relatório, conhece o processo e
age sobre a causa. O módulo do assunto tinha inventado um cargo para conseguir
existir.

**O mesmo erro, noutro sistema.** Num atendimento dividido por departamentos,
`papel = 'supervisor'` na tabela de usuários respondia "esta pessoa
supervisiona?" quando a pergunta real é sempre "supervisiona **isto aqui**?".
Como a resposta global era sim, valia em todo departamento — inclusive nos que a
pessoa não atendia ([[Supervisão é papel do setor, não cargo global]]).

Duas formas do mesmo engano: o recorte global inventa um papel; o recorte por
área usa o papel que já existe no organograma.

## Na prática

- **O teste é nomear a pessoa.** "Quem abre esta tela?" Se a única resposta é um
  cargo que passou a existir por causa do software, o recorte está errado. O
  gestor do Fiscal existe com ou sem sistema; "a coordenação de post mortem do
  escritório" não.
- **O que fica comum é a implementação, não a casa.** As quatro áreas
  compartilham o formulário, a consulta e a regra — escritos uma vez; cada uma
  tem a sua porta, e é a porta que carrega a permissão
  ([[O que dois módulos compartilham é a query, não a rota]]).
- **Área sem casa ganha casa.** Se um setor participa do assunto e não tem
  módulo, o módulo nasce — mesmo com uma seção só. Módulo de uma seção é começo,
  não defeito; sem ele não existe gestor daquela área, e o assunto volta a ser
  lido por quem não é dela.
- **Perder a visão que cruza áreas é o preço, e costuma ser barato.** Quem quer
  o retrato do conjunto quer um relatório, não uma tela de operação — e um
  relatório se monta depois, sobre o mesmo dado, sem precisar que a operação
  inteira more junta.
- **Cuidado ao converter a permissão.** Quem tinha a visão global lia tudo:
  desligar isso na migration é tirar acesso de gente que trabalhava. Converta
  para o equivalente mais largo (a gestão de cada área) e deixe o ajuste fino
  para a tela de cargos — SQL não sabe quem deve ficar com qual área.

## Conexões
- Irmã: [[A casca se compartilha por público, não por marca]]
- Técnica que aplica: [[Supervisão é papel do setor, não cargo global]] · [[O que dois módulos compartilham é a query, não a rota]] · [[Posse numa permissão binária é duas seções e recorte por linha]]
- Visto em: [[Navetech Hub]] · [[Navehub]]
- Mapa: [[Base]]
