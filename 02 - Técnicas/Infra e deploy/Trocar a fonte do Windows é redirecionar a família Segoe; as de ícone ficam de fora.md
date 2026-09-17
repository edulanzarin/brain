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
"Segoe UI"="Inter"
"Segoe UI Light"="Inter Light"
"Segoe UI Semilight"="Inter Light"
"Segoe UI Semibold"="Inter SemiBold"
"Segoe UI Black"="Inter Black"
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
distribui em famílias GDI separadas, e o formato varia entre fontes. A Inter
agrupa Regular/Bold/Italic/BoldItalic em `Inter` e põe cada peso extra em família
própria (`Inter Light`, `Inter SemiBold`) — o mesmo formato da Segoe, e é isso
que faz negrito e itálico continuarem resolvendo certo. Ler o `name` id 1 de cada
`.ttf` antes de escrever a tabela leva dez linhas e evita descobrir pela tela.

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
