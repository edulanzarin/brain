---
tags: [tipo/atomica, camada/principio, dados]
criado: 2026-09-17
---

# Leitura extraída se prova pela redundância que o próprio documento imprime

> Documento de fechamento diz a mesma coisa mais de uma vez: o saldo de cada linha encadeia o da anterior, a linha soma no total do grupo, o grupo soma no total geral, o residual é valor menos depreciação. Essa redundância é um gabarito que já vem dentro do arquivo. Leitura que não se confere contra ele (parser, OCR, LLM) pode errar calada; leitura que se confere não consegue.

## Por que importa

Extração erra de um jeito típico: pega a coluna vizinha. O número lido é plausível, tem o formato certo, e nada nele sozinho denuncia o erro. Só a relação entre números denuncia. Um relatório de bens colado no ChatGPT voltou com o valor residual no lugar da depreciação acumulada a partir da quinta linha, e ninguém percebeu olhando a planilha. Contra o total da conta, que o relatório imprime, a soma deixa de bater na hora.

## Como aplicar

- **Antes de escrever o leitor, liste as identidades que o documento imprime**: saldo anterior + movimento = saldo; Σ linhas = total do grupo; Σ grupos = total geral; valor − depreciação = residual; Σ devedores = Σ credores.
- **Leia os totais também.** Eles não são ruído a descartar, são o dado de conferência.
- **A identidade também desempata colunas.** Quando duas colunas costumam ser iguais (valor e valor corrigido, depreciação acumulada e do período), escolha a que fecha a identidade em vez de confiar na posição ou na ordem de desenho. Quando divergem, só a certa fecha.
- **Mostre a conferência na tela, persistente**, e recalcule quando o humano edita. Confiança silenciosa não vale nada; "confere com o total impresso" vale.
- **Onde o documento não imprime redundância, a leitura não tem prova.** Diga isso em vez de afirmar.

O limite: o total fechar não garante que cada linha está no grupo certo. Um erro que se compensa entre grupos passa pelo total geral, e só a conferência por grupo (ou por registro) o pega. Ver a irmã [[Auditar o registro, não só o agregado]].

## Casos

- **Extrato bancário**: a cadeia de saldos fecha do saldo inicial ao final, e ainda dá o sinal de cada lançamento. Em [[Ler extrato bancário em PDF]].
- **Relatório de bens do imobilizado**: residual por bem, total por conta e total geral. Em [[Relatório com registro em várias linhas se lê na ordem de desenho do PDF]].
- **Balancete de abertura**: Σ devedores = Σ credores; se não fecha, o PDF veio mal lido antes de qualquer importação.

## Conexões
- Irmã: [[Auditar o registro, não só o agregado]]
- Irmã: [[Coleta determinística, LLM só interpreta]]
- Visto em: [[Navetech Hub]] (Conciliação, Implantação de saldos e do patrimonial)
- Mapa: [[Base]]
