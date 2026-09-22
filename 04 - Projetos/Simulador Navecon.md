---
tags: [tipo/projeto, projeto/simulador-navecon]
criado: 2026-09-22
status: ativo
codigo_em: ~/Dev/simulador-navecon
---

# Simulador Navecon

> Simulador tributário de captação: um quiz de nove perguntas estima quanto a
> empresa paga de imposto **além do devido** e só entrega o resultado depois do
> cadastro. Vai ser apresentado numa plateia de 600+ pessoas.

Código em: `~/Dev/simulador-navecon`

## Estado atual

No GitHub, privado: **`git@github.com:edulanzarin/simulador-navecon.git`**. Build
limpo, 46 testes verdes, telas conferidas por print headless em celular e
desktop.

**Roda de pé na máquina** (22/09/2026): `docker compose up -d --build` sobe
app + db + migrate, e o fluxo foi exercitado ponta a ponta — lead gravado pela
API, diagnóstico recalculado no servidor batendo com o esperado (17,60%,
R$ 1.056.000 de carga, faixa R$ 52.377–84.480, índice 72), painel listando,
CSV exportando e corpo inválido voltando 400 com a lista de problemas.

**Não subiu para o servidor**, e o modelo tributário ainda está com os números
de partida, não com os da Navecon.

Vai rodar em **`simulador.navecon.net.br`**, `APP_BASE_PATH=/`.

## Quem atende a 443 decide o compose

O compose de produção daqui publica 80/443 com Caddy próprio, e o do
[[Evento Navecon]] faz igual. **Os dois no mesmo host colidem** com
`port is already allocated`, e o erro só aparece depois do build, com o DNS já
apontado.

O TI deu acesso a um servidor onde `docker compose up -d --build` puro já
responde no domínio, o que significa proxy reverso no host. Nesse cenário o
simulador **não sobe Caddy**: publica em `127.0.0.1:4082` e o TI aponta o vhost.
O que precisa ser pedido junto é o repasse de `X-Forwarded-For` e
`X-Forwarded-Proto` — sem eles o `trust proxy` do Express vê a plateia inteira
como um IP só no rate limit, e o cookie `Secure` do `/admin` não fecha o login.

Os três comandos que descobrem o cenário estão no README do projeto.
**Falta confirmar qual é**, e é a última incógnita técnica do deploy.

## De onde veio

O Fábio viu a calculadora da **Alvus** (concorrente de planejamento tributário)
num reel do Instagram e pediu uma equivalente com a cara da Navecon. A página
deles é um HTML único com o JavaScript na mão, então o modelo inteiro estava
legível: nove perguntas, muro de cadastro, painel de resultado, e o lead indo
para uma planilha via Google Apps Script.

## Infra

Slug `simulador-navecon` · app `simulador-navecon-app` na `4082` · banco
`simulador-navecon-db` na `5082`. Migrations em `simulador-navecon-migrate`.
Chassi e mapa de portas em [[Infra]].

## Stack

**Node/Express (TS, rodado com tsx)** servindo o SPA buildado e a API na mesma
origem · **React/Vite** · **Postgres 16** · **nodemailer** com SMTP do Gmail ·
**Caddy** com HTTPS automático no compose de produção. Sem Git LFS: o simulador
não tem mídia pesada, então a armadilha das mídias sumirem no deploy (que morde
o [[Evento Navecon]]) não existe aqui.

## Decisões importantes

- **As tabelas tributárias moram num arquivo só.** Todo número com opinião
  fiscal vive em `modelo/tabelas.ts`, comentado para ser revisado por quem
  entende de imposto e não de código. São cinco tabelas (carga por segmento ×
  regime, fator de excesso, direção da Reforma, índice do devido, adequação do
  regime). Instância de
  [[A definição em dado dirige o comportamento, não um caso no código]].
- **O modelo fica fora de `src/`, na raiz.** `modelo/` é importado pelo
  navegador E pelo servidor, porque o servidor **recalcula** o diagnóstico a
  partir das respostas e ignora o número que o cliente mandou. O `Dockerfile`
  copia `modelo/` junto de `server/` no estágio de runtime.
- **Muro de cadastro, não login.** Nome, empresa, WhatsApp, e-mail e quem cuida
  da contabilidade hoje, sem senha e sem confirmação por e-mail. A plateia
  preenche de pé, no celular, com o sinal do auditório: cada etapa a mais é
  gente que não chega ao resultado. Senha existe só no `/admin`, do nosso lado.
- **O resultado não fica refém do envio.** Depois de duas falhas de rede aparece
  "ver o resultado assim mesmo". Ver
  [[A entrega não fica refém do registro que pode falhar]].
- **Postgres, não planilha.** O lead da Alvus vai para o Apps Script com
  `mode: "no-cors"`, que não devolve resposta: destino fora do ar perde o
  contato em silêncio. Aqui é banco, com aviso por e-mail best-effort por cima.
- **Guarda o diagnóstico que foi mostrado, não só as respostas.** As tabelas vão
  mudar quando o Fábio recalibrar, e sem isso seria impossível saber qual número
  cada lead viu. Colunas para filtrar e exportar, mais o JSON cru para
  reprocessar.
- **Rate limit generoso no POST de lead** (300 por 10 min). A plateia inteira sai
  do mesmo Wi-Fi e o Express enxerga um IP só; apertar ali bloquearia todo mundo
  depois dos primeiros envios.
- **Painel `/admin` server-rendered**, mesmo arranjo do [[Evento Navecon]]:
  HTML gerado no Express, sem JS no cliente (a CSP só libera `script 'self'`),
  ações por form POST, sessão em cookie assinado. Lista com filtro por status,
  por direção da Reforma e busca; CSV com `;` e BOM.
- **Nasce com catálogo.** `/sistema` com as peças vivas e `/sistema/previa` com
  o resultado inteiro em três perfis de mentira. Foi o que permitiu julgar cada
  estado por URL, sem preencher o quiz a cada print. Ver
  [[Catálogo de componentes é contrato vivo, não documentação]].
- **Uma etapa por tela, centrada na altura.** Cada fase é um palco de `100dvh`
  numa coluna de 440px, com o progresso grudado no alto e a ação grudada
  embaixo, no alcance do polegar. A primeira versão jogava tudo contra o topo e
  contra as bordas do celular, e as nove opções da pergunta de segmento não
  cabiam numa tela. Ver
  [[Centralizar na altura é margin auto, porque justify-content corta o topo]].

## O defeito que veio junto no modelo

O modelo de origem descontava pontos da carga quando a margem era apertada,
inclusive no Lucro Presumido. Como no Presumido a base é **arbitrada em lei**,
margem apertada significa pagar sobre lucro que não existiu: a situação piora,
não melhora. O sinal estava trocado, e a consequência visível era uma empresa
faturando R$ 400.001/mês aparecer pagando **menos** imposto no ano do que uma
faturando R$ 400.000, ao cruzar o teto do Simples.

Corrigido com um piso no ajuste (no Presumido a margem só agrava) e trancado por
teste de propriedade. Ver
[[Quando o degrau é real, preserve a monotonia em vez de suavizar]].

## Nome e domínio

`simulador.navecon.net.br`. Escolhido contra `diagnostico`, `calculo` e
`reforma` por um critério só: **sobreviver a ser ouvido no palco e digitado no
celular**. Nove letras, sem acento, sem hífen, sem armadilha de plural. O
posicionamento que "diagnóstico" daria foi recuperado na copy, que chama a peça
de Diagnóstico Tributário Navecon.

## Aprendizados (viraram notas)

Só links. O texto mora na nota de técnica/princípio.

- [[Quando o degrau é real, preserve a monotonia em vez de suavizar]]
- [[Centralizar na altura é margin auto, porque justify-content corta o topo]]
- [[Media query mede a janela; quem decide a quebra é a largura do contêiner]]
- [[A entrega não fica refém do registro que pode falhar]]
- [[No Windows, duas coisas escutam a mesma porta e o cliente fala com a errada]]
  (ganhou a variante IPv4/IPv6 disputando a mesma porta)
- [[Print headless pelo navegador já instalado fecha o ciclo de julgar a tela]]
  (ganhou a armadilha do `--window-size`, que dimensiona a captura e não o layout)

## Próximos passos

- [ ] **Fábio revisar `modelo/tabelas.ts`.** É o bloqueio real antes do palco: os
      números atuais são os do concorrente, não os da carteira da Navecon.
- [x] Subir a stack local e validar o fluxo ponta a ponta (22/09/2026)
- [ ] **Descobrir quem atende a 443 no servidor** (três comandos no README): decide
      se sobe com Caddy próprio ou se o TI aponta um vhost para 127.0.0.1:4082
- [ ] DNS A/AAAA de `simulador.navecon.net.br` no IP do servidor
- [ ] Preencher `.env` de produção: `POSTGRES_PASSWORD`, `ADMIN_USER`,
      `ADMIN_PASSWORD`, SMTP
- [ ] Confirmar o WhatsApp de atendimento (hoje usa o do rodapé da imersão,
      `47 9237-0273`)
- [x] Repositório no GitHub, privado (22/09/2026)

## Conexões
- Usa: [[Design]] · [[Infra]]
- Mapa: [[Projetos]]
