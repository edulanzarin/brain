---
tags: [tipo/projeto, projeto/navex]
criado: 2026-09-24
status: ativo
codigo_em: C:/Dev/navex
---

# NaveX

> A plataforma de trabalho da Navecon sobre o banco do Questor, refeita do zero
> com cara nova. Substitui o [[Navetech Hub]] (o Nexo): mesma função, mesma
> camada de domínio, interface inteira nova.

Código em: `C:/Dev/navex` (sem remote ainda).

## Por que existe

Em 24/09/2026 o Eduardo pediu para refazer "completamente TUDO com design mais
moderno", sem copiar a reescrita anterior do Nexo (a pasta `nexo`, que ele chama
de nexo1 e não aprovou), com todas as funções do nexo2 (o que está em produção).
Nome novo: NaveX. Ordem: Contábil primeiro.

## Estado atual

- Esqueleto, sistema de design, catálogo `/sistema` com prévia de tela, login,
  início e a moldura do módulo prontos.
- **Contábil em construção**: as quinze seções do nexo2 sendo montadas sobre os
  primitivos novos.
- Fiscal, DP, RH, Obrigações, Societário, Configurações e a administração
  (usuários, cargos, grupos) seguem no Nexo.

## Infra

Slug `navex` · `navex-app` na **4083** · `navex-db` na **5083** · migrations no
serviço `navex-migrate`. Chassi em [[Infra]]. Em desenvolvimento, o banco roda
num cluster Postgres portátil na 5083
([[Sem virtualização na BIOS não há Docker no Windows; o banco de dev vira Postgres portátil]]),
porque a máquina não aguenta Docker Desktop junto com o resto.

## Decisões importantes

- **Domínio portado, interface nova.** `src/lib` (o SQL do Questor e os motores),
  as rotas de API e as 39 migrations vêm do nexo2 como estão. O schema é
  idêntico de propósito: no dia da troca, o NaveX assume o banco do app do
  nexo2 sem migrar dado. Os ids de seção também são os mesmos, porque são a
  chave de `cargo_secao`.
- **Três furos do nexo2 nascem fechados**: replicar plano de contabilização e
  apagar regra de extrato conferem o escopo de empresa, e o aprendizado de CFOP e
  de conta efetiva tem trava por empresa
  ([[Regravar o conjunto de uma chave com delete e insert exige trava por chave]]).
- **Contexto de trabalho no topo.** Empresa, grupo, filial e período moram na URL
  e valem para todas as seções do módulo; a tela não tem barra de filtro própria
  para eles. Cada aba declara no catálogo o que usa (período por dia, por mês ou
  nenhum; empresa obrigatória, opcional ou nenhuma; filial; verbo do botão).
- **Execução por botão na moldura.** A tela só monta depois da primeira execução
  (ou só com empresa, na bancada), e quando o recorte muda depois da execução o
  resultado continua na tela com o aviso de que mudou
  ([[Consulta pesada executa por botão, não por mudança de filtro]]).
- **Visual**: vidro sobre papel quadriculado que se apaga para baixo, dois temas
  (noite e dia), laranja da marca só na ação, azul para foco e seleção,
  Instrument Sans com eixo de largura (número de indicador a 82%), faixa de
  indicadores em vez de grade de cartões, paleta Ctrl+K com seções e empresas.
  A marca é um X de dois traços, laranja e azul, cruzando: a conferência entre
  duas fontes que o sistema faz o dia inteiro.

## Aprendizados (viraram notas)

- [[Escala própria do tema precisa ser ensinada ao tailwind-merge]]
- [[Fundo no body cobre a camada de z-index negativo]]

## Próximos passos

- [ ] Fechar as quinze seções do Contábil e validar contra o Questor na rede do escritório.
- [ ] Administração (usuários, cargos, setores, grupos) e perfil.
- [ ] Fiscal, DP, RH, Obrigações, Societário e Configurações.
- [ ] Remote no GitHub e deploy no servidor.

## Conexões
- Substitui: [[Navetech Hub]]
- Usa: [[Design]] · [[Infra]] · [[Banco Questor]]
- Mapa: [[Projetos]]
