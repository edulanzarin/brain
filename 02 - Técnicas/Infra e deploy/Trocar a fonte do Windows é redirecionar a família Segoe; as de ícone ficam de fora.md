---
tags: [tipo/atomica, camada/padrao, infra, armadilha]
criado: 2026-09-17
---

# Trocar a fonte do Windows é redirecionar a família Segoe; as de ícone ficam de fora

> A troca não instala fonte nova "por cima": esvazia os ponteiros da `Segoe UI`
> no registro e manda a família para outra. Funciona porque os `.ttf` continuam
> no disco — e quebra se levar junto as famílias que desenham ícone.

## O problema

A fonte de interface do Windows não é configurável por tela. Só existe o caminho
do registro, em dois ramos de
`HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion`:

- `Fonts` — o mapa nome da fonte -> arquivo `.ttf`.
- `FontSubstitutes` — o mapa nome pedido -> nome entregue, consultado pelo GDI
  antes de procurar a fonte instalada.

Os tutoriais resumem isso como "apague a Segoe e aponte pra sua fonte". É onde a
coisa quebra: `Segoe UI` parece uma fonte e são dezesseis entradas com papéis
diferentes.

## A solução

**Esvaziar o valor, não apagar o arquivo.** As doze entradas de texto da família
viram string vazia, o que tira a fonte da mesa sem remover o `.ttf` do disco.
Reverter é reescrever os nomes de arquivo — nada foi destruído.

```
[HKLM\...\CurrentVersion\Fonts]
"Segoe UI (TrueType)"=""            ; e as outras onze de texto

[HKLM\...\CurrentVersion\FontSubstitutes]
"Segoe UI"="Roboto"
"Segoe UI Light"="Roboto Light"
"Segoe UI Semilight"="Roboto"        ; Regular, nao Light -- ver abaixo
"Segoe UI Semibold"="Roboto Medium"
"Segoe UI Black"="Roboto Black"
```

**Quem fica de fora, e por quê:**

| Família | Por que não se toca |
|---|---|
| `Segoe Fluent Icons`, `Segoe MDL2 Assets` | desenham os ícones da interface. Substituir apaga seta, wi-fi, bateria e o botão de fechar |
| `Segoe UI Emoji`, `Segoe UI Symbol`, `Segoe UI Historic` | nenhuma fonte de texto cobre esse intervalo Unicode |
| `Segoe UI Variable` | é o que o Win11 usa nos apps WinUI (Configurações, Explorer). Carrega forma, mas responde a outro caminho de código: é a parte que mais quebra espaçamento e a que o Windows Update reescreve |

Deixar a `Variable` de fora tem preço: os apps WinUI seguem em Segoe, e a
interface fica em duas fontes. É uma troca consciente de consistência por risco.

**Os pesos saem da tabela `name`, não de chute.** Uma fonte com muitos pesos os
distribui em famílias GDI separadas, e o formato varia entre fontes. A Roboto
agrupa Regular/Bold/Italic/BoldItalic em `Roboto` e põe cada peso extra em família
aparte (`Roboto Light`, `Roboto Medium`, `Roboto Black`) — o mesmo formato da Segoe,
e é isso que faz negrito e itálico continuarem resolvendo certo. Ler o `name` id 1 de
cada `.ttf` antes de escrever a tabela leva dez linhas e evita descobrir pela tela.

**O peso ausente arredonda pra baixo, não pra cima.** A Segoe tem Semilight (350) e
Semibold (600); a Roboto não tem nenhum dos dois. Semilight vai pra Regular (400) e
não pra Light (300): mandar pra Light deixa fino demais boa parte do texto do Win11,
que é onde o Semilight é usado. Semibold vai pra Medium (500), porque Bold (700) pesa
demais num rótulo de interface.

## Escolher a substituta é medir, não olhar

Uma fonte bonita não é uma fonte que cabe. O Windows desenha menu, diálogo, lista e
botão contando com as **larguras de avanço da Segoe**; a substituta herda esse espaço
já reservado. Se ela for mais larga, tudo estoura, corta com reticências e aperta --
sem nenhum erro em lugar nenhum.

Medir é rápido: registrar cada candidata só no processo com `AddFontResourceEx` e
`FR_PRIVATE` (não instala nada) e comparar `MeasureText` da mesma frase contra a Segoe.
O x-height sai do `OS/2` dividido pelo `unitsPerEm` do `head`.

| | x-height | largura da mesma frase |
|---|---|---|
| Segoe UI | base | base |
| Inter | +9,2% | **+8,8%** |
| Roboto | +5,7% | +0,5% |
| Open Sans | +7,0% | +0,5% |
| IBM Plex Sans | +3,2% | +0,5% |
| Source Sans 3 | -2,8% | +0,5% |
| Selawik | 0% | +0,5% |

A leitura que importa: **largura e x-height são eixos separados**. Quase toda fonte de
interface larga usada hoje foi ajustada pra caber no mesmo avanço, então a coluna da
direita é praticamente constante -- e a Inter é a exceção que quebra. Escolher pela
coluna do x-height é escolher o quanto a interface parece cheia; escolher pela largura
é escolher se ela funciona.

O segundo eixo é **hinting**. Fonte desenhada pra tela densa com antialiasing cinza (a
Inter é o caso típico) fica fina e lavada em 1080p com ClearType, e pior ainda em tema
escuro, onde texto claro sobre fundo escuro borra pra fora. Fonte que nasceu pra tela
fraca -- Segoe, Roboto, Selawik -- tem hinting manual ou forte e aguenta. Não dá pra
julgar isso num navegador: `TextRenderer` desenha pelo mesmo GDI dos controles, e a
prova de que o ClearType está ativo é achar pixel com franja colorida no resultado.

## O que mais vale lembrar

- **O desfazer se gera do estado lido, antes da mudança.** Um `reg export` dos
  dois ramos, mais um `.reg` que repõe exatamente os nomes de arquivo que estavam
  lá. Desfazer escrito de memória ou de tutorial é chute com cara de rede.
- **Verificar a instalação antes de mexer na Segoe.** Se a fonte nova não
  registrou e a Segoe já saiu, sobra Tahoma. Conferir as famílias exigidas e
  abortar antes do passo destrutivo é barato.
- **O Modo de Segurança usa fonte própria.** É a saída se a interface ficar
  ilegível — dá pra aplicar o `.reg` de lá.
- **Volta sozinho no Windows Update.** O update reinstala a Segoe e reescreve o
  ramo `Fonts`. As fontes novas continuam instaladas; só a substituição se perde,
  e reaplicar o `.reg` resolve.
- **A substituição vale também no DirectWrite.** Chromium e Electron pedem
  `Segoe UI` e recebem a nova — ou seja, VS Code e afins herdam a troca sem
  configuração. Editor e terminal, porém, precisam de monoespaçada à parte.
- **Instalar fonte é copiar para `C:\Windows\Fonts` e registrar.** O nome do valor
  é o nome completo (`name` id 4) mais ` (TrueType)`, e o dado é só o nome do
  arquivo. Exige elevação.

## Conexões
- Princípio: [[Substituição global se decide membro a membro, não pelo nome da família]]
- Irmã: [[Formatar a máquina perde tudo que o git não versiona]]
- Mapa: [[Infra]]
