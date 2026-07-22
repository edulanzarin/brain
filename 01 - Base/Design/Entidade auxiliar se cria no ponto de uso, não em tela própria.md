---
tags: [tipo/atomica, camada/principio, design]
criado: 2026-07-22
---

# Entidade auxiliar se cria no ponto de uso, não em tela própria

> Coisa que só serve para organizar outra coisa nasce e se gerencia de onde já é
> usada — não ganha aba, rota nem CRUD próprio.

## A regra

Uma entidade **auxiliar** (existe só para rotular, agrupar ou anexar a entidade
principal — um grupo de empresas, uma etiqueta, uma categoria) não é um destino de
navegação. Ela é um atributo. Então:

- **Cria-se onde é escolhida**: digitar um nome que não existe no próprio seletor
  já a cria. Nada de "vá primeiro à tela X, cadastre, volte e selecione".
- **Gerencia-se onde aparece**: renomear e excluir moram no mesmo seletor/filtro
  onde ela é lida.
- **Some sozinha quando esvazia**: sem nenhuma entidade principal apontando pra
  ela, é varrida automaticamente — ninguém faz faxina de registro auxiliar à mão.

Se sobrou uma aba só para ela, a aba está pagando por um modelo mental errado: o
auxiliar virou "coisa" quando devia ser "campo".

## Por que

A tela própria cobra um pedágio de navegação em toda criação: o fluxo natural
(estou cadastrando a empresa / subindo o certificado) trava porque o grupo ainda
não existe, e o usuário tem que sair, criar noutro lugar e voltar. Pior quando a
entidade principal é criada **automática** (o certificado sobe e puxa a empresa
sozinho): o auxiliar não tem por onde entrar sem uma segunda ida manual.

No Cofre Digital havia uma aba Grupos só para isso. Ela sumiu: o grupo passou a
nascer no formulário da empresa e do certificado (digitando o nome), e a
renomear/excluir dentro do próprio filtro de grupo. Grupo sem empresa é podado no
servidor. Mesmo instinto do editor de manual do cofre, onde a imagem entra por
Ctrl+V no ponto do texto em vez de uma galeria à parte.

## Na prática

- O seletor do auxiliar oferece "Criar «nome»" quando o texto digitado não casa com
  nenhum item — ver [[Seletor cria e gerencia os próprios itens]].
- A poda de vazios roda no backend depois de qualquer mutação que possa esvaziar
  (troca de vínculo, exclusão da principal), não num botão de limpeza.
- Guardar a permissão certa: criar/renomear/excluir o auxiliar é ação da entidade
  **principal** (mexe no cadastro dela), mesmo quando disparada de outra tela.

## Conexões
- Irmã: [[Estado de tela pertence à seção, não à página]]
- Técnica que aplica: [[Seletor cria e gerencia os próprios itens]]
- Visto em: [[Cofre Digital]]
- Mapa: [[Base]]
