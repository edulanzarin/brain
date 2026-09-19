---
tags: [tipo/atomica, camada/padrao, infra, docker, armadilha]
criado: 2026-09-19
---

# Com o hypervisor do Windows no ar, o WMI diz que a CPU não virtualiza

> `Win32_Processor.VirtualizationFirmwareEnabled` responde `False` sempre que o
> hypervisor do Windows já está rodando, e no Windows 11 ele roda desde o primeiro
> boot. A pergunta certa é se há hypervisor: se há, a BIOS virtualiza.

## O problema

O teste que aparece em toda resposta sobre Docker no Windows é este:

```powershell
(Get-CimInstance Win32_Processor).VirtualizationFirmwareEnabled
```

Com o hypervisor no ar, o próprio Windows passa a rodar como uma partição dentro
dele, e a CPU que ele enxerga não expõe mais a extensão de virtualização. A flag lê
`False` com VT-x ligado na BIOS.

E o Windows 11 liga o hypervisor sozinho: a Integridade de Memória (HVCI, parte da
segurança baseada em virtualização) vem ativa por padrão numa instalação limpa em
hardware compatível. A máquina recém-formatada, que é onde alguém roda esse teste,
é justamente a que ele engana.

Numa instalação de 11/09/2026, esse `False` virou "a BIOS não virtualiza", e por uma
semana o [[Navetech Hub]] e o [[telebot]] subiram banco de dev em Postgres portátil
([[Sem virtualização na BIOS não há Docker no Windows; o banco de dev vira Postgres portátil]]).
O log do sistema mostrava o hypervisor iniciando em todo boot desde a instalação.
Quando o Docker Desktop enfim foi instalado, subiu no primeiro reinício: a BIOS
estava ligada o tempo todo.

## A solução

Perguntar pelo hypervisor antes de perguntar pela CPU:

```powershell
(Get-CimInstance Win32_ComputerSystem).HypervisorPresent          # True = virtualiza
(Get-CimInstance Win32_Processor).VirtualizationFirmwareEnabled   # só vale se o de cima for False
```

Hypervisor rodando prova a virtualização, porque ele não sobe sem ela. Sem
hypervisor, a flag do processador volta a dizer a verdade. Outras leituras que não
caem na armadilha:

- `systeminfo`: com hypervisor, a seção do Hyper-V diz "Hipervisor detectado" em vez
  de listar os requisitos.
- Gerenciador de Tarefas, Desempenho, CPU: "Virtualização: Habilitado".
- Log do sistema, fonte `Microsoft-Windows-Hyper-V-Hypervisor`, evento 1: um por boot
  em que o hypervisor subiu. É a única que responde pelo passado ("virtualizava na
  semana passada?").

## O que mais vale lembrar

- **O `wsl --status` também culpa a BIOS sem ser ela.** Logo depois do
  `wsl --install`, antes de reiniciar, ele diz que o WSL2 não inicia "porque a
  virtualização não está habilitada nesta máquina". O que falta é a Plataforma de
  Máquina Virtual, que só ativa no boot.
- A prova mais barata é tentar: WSL e Docker Desktop instalam sem virtualização, e o
  reinício que o instalador pede responde o resto. São dez minutos; o desvio custou
  uma semana.
- Se a mesma forma aparecer em outro assunto (um sinal indireto diz "não dá" e o
  plano desvia sem ninguém tentar), ela vira princípio: sinal que proíbe um caminho
  se confirma tentando o caminho.

## Conexões
- Princípio: [[Contador que conta sucesso de promessa afirma que deu certo]] (a
  evidência errada manda procurar no lugar errado, e com confiança)
- Irmã: [[Sem virtualização na BIOS não há Docker no Windows; o banco de dev vira Postgres portátil]]
- Visto em: [[Navetech Hub]] · [[telebot]] · [[Privello]]
- Mapa: [[Infra]]
