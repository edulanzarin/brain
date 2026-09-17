---
tags: [tipo/atomica, camada/padrao, dev/backend]
criado: 2026-09-17
---

# Relatório com registro em várias linhas se lê na ordem de desenho do PDF

> `pdftotext -layout` junta o texto por altura na página, o que serve para tabela de uma linha por registro. Quando cada registro é um bloco de várias linhas com colunas empilhadas (o relatório de bens do SCI), o `-layout` cola o valor de um bem no cabeçalho do vizinho. O `-raw` segue a ordem em que o PDF desenha, e gerador de relatório desenha registro por registro: cada bloco sai inteiro.

## O problema

No "Correção e depreciação" do SCI, cada bem tem cabeçalho (`13 - 21/10/2024 - COMIN COMERCIO DE MOVEIS`), fornecedor, documento e duas linhas de colunas. Com `-layout`, a depreciação do bem 13 saiu numa linha ACIMA do cabeçalho dele, dentro do bloco do bem 3. Nenhuma heurística por linha conserta isso: a leitura por altura já misturou os registros.

## A solução

Extrair em `-raw` e ler por marcos e tokens:

- **Marcos por linha**: cabeçalho do grupo (`Conta: 102 - 01.2.3.01.005 - Veículos`), cabeçalho do registro, total do grupo, total geral. O corpo de cada marco é o texto até o próximo.
- **Valores por token, não por linha.** xpdf e poppler seguem a mesma ordem de desenho, que é do arquivo, mas podem juntar ou quebrar linhas de outro jeito. O teste que prova: juntar todas as linhas numéricas numa só e separar cada valor numa linha própria têm que dar a mesma leitura.
- **Âncora antes de contar colunas.** O bloco tem rótulo com número que parece dinheiro (`Valor moeda: 0,21`); as colunas só começam depois do último rótulo (`Saldo em quantidade: 0`).
- **Dinheiro com duas casas que não casa pedaço de outro número**: lookbehind/lookahead contra dígito, ponto e vírgula, para não pegar o `1.458.020,33` de dentro de `1.458.020,330000`, e fora a taxa seguida de `%`.
- **A ordem de desenho não é a da tela, e o rótulo nem sempre vem antes do valor.** O total geral do SCI é desenhado antes de "TOTAL GERAL DO RELATÓRIO", colado nos valores do total da última conta; o leitor pega os cinco primeiros como total da conta e os cinco seguintes como total geral.
- **Coluna ambígua se decide pela identidade**, não pela posição: valor e valor corrigido, depreciação acumulada e do período vêm iguais quase sempre; escolhe-se o par que fecha valor − depreciação = residual.

## Armadilhas de ambiente

- **xpdf ≠ poppler na borda.** O `pdftotext` do Git for Windows é xpdf: sai em Latin-1 por padrão e não lê PDF por stdin (`-` só vale como saída). O do container é poppler: UTF-8 e stdin. Passe `-enc UTF-8` sempre; em teste local, passe o caminho do arquivo.
- **Fixture**: o texto `-raw` do PDF real com identificação trocada (empresa, CNPJ, cidade, escritório, placa) e os valores reais, que são o que prova a leitura.

## Conexões
- Princípio: [[Leitura extraída se prova pela redundância que o próprio documento imprime]]
- Irmã: [[Ler extrato bancário em PDF]]
- Depende de: [[Armadilhas de child_process no Node]]
- Visto em: [[Navetech Hub]] (Implantação do patrimonial; extrato do Ailos na Conciliação, onde o `-layout` jogava valor e saldo para as linhas de baixo e o `-raw` devolve uma linha por lançamento)
- Mapa: [[Backend]]
