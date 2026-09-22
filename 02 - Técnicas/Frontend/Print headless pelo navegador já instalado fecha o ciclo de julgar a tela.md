---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-09-17
---

# Print headless pelo navegador já instalado fecha o ciclo de julgar a tela

> Escrever tela sem ver a tela é chutar. Não precisa de Playwright nem de
> serviço: o Edge (ou Chrome) que já está na máquina abre em modo invisível com
> uma porta de depuração, e por ela dá para logar com cookie, medir a página e
> salvar o PNG. Vinte linhas, zero dependência nova, e o print volta para quem
> escreveu o CSS.

## O caminho

```
msedge --headless=new --remote-debugging-port=9333 --user-data-dir=<tmp>
       --disable-gpu --hide-scrollbars --window-size=1440,900 about:blank
```

Depois, por HTTP e WebSocket (o `WebSocket` global do Node 22 basta):

1. `PUT /json/new?about:blank` devolve um alvo NOVO com o endereço do socket.
2. `Network.setCookie` com `url` (não `domain`) planta a sessão.
3. `Emulation.setDeviceMetricsOverride` fixa a viewport.
4. `Page.navigate`, esperar `Page.loadEventFired`, esperar mais um pouco pela
   animação de entrada.
5. Página inteira: `Page.getLayoutMetrics` dá a altura do conteúdo, que vira nova
   altura de viewport antes de `Page.captureScreenshot`.

## Detalhe fino precisa de escala, não de print maior

O print da página inteira responde "a tela está de pé". Ele não responde "essa
pista de 14px aparece?", porque a imagem é reduzida para caber na tela de quem
julga, e o detalhe some na redução — o que parece ausência de bug é perda de
resolução.

Duas saídas, e o `clip` do CDP faz as duas: `scale: 2` na página inteira, ou um
recorte por elemento (`DOM.getBoxModel` → `clip`) em dobro. A segunda é a que
serve para julgar tipografia e contraste de um componente.

E quando o seletor "não casa com nada", desconfie do servidor antes do seletor:
a página de erro do navegador não tem `main` nem as classes do app, então um
servidor caído se disfarça de seletor errado.

## Quatro armadilhas que custam meia hora cada

- **`--window-size` dimensiona a captura, não o layout — e mente calado.** É a
  pior das quatro, porque o PNG sai com exatamente a largura pedida. Pedindo
  390x844 para julgar celular, o arquivo tem 390px e parece certo; por dentro a
  página foi diagramada nos 800px padrão do headless e o print é um RECORTE
  dela. O sintoma é uma tela que parece estourada — título cortado, grade em
  duas colunas onde deveria ter uma — e o defeito não existe. Vale nos dois
  modos, `--headless=new` e `--headless` antigo. Quem muda a largura que o CSS
  enxerga é só o `Emulation.setDeviceMetricsOverride` do passo 3, e com ele o
  `--window-size` deixa de importar. Regra prática: se o layout responsivo não
  reagir à largura pedida, é isto, não o CSS.
- **O primeiro alvo pode não ser a sua página.** Num perfil zerado, o Edge abre
  um diálogo próprio ("sincronizar seus dados"), e `/json/list` entrega ele. Daí
  o print sai de uma tela que não é a sua. Criar o alvo resolve.
- **Sem `--disable-gpu`, o print sai chapado** numa máquina sem aceleração
  disponível para o processo headless.
- **No Git Bash, argumento que começa com `/` vira caminho do Windows.** Passar a
  rota `/` para o script chega como `C:/Program Files/Git/`, e o CDP responde
  "Cannot navigate to invalid URL". `MSYS_NO_PATHCONV=1` na frente do comando.

## Por que vale

O ciclo vira: escreve, builda, imprime, OLHA, corrige. Foi olhando o PNG que
apareceram o cartão esticado com buraco no meio, a arte espalhada demais e a
faixa vazia entre filtro e conteúdo — nenhum dos três aparece lendo o JSX. Serve
também para conferir o que só existe em tamanho pequeno (celular) sem ter o
aparelho na mão.

## Conexões
- Princípio: [[Verificar no build de produção, não só em dev]]
- Irmã: [[Armadilhas de child_process no Node]]
- Visto em: [[Navetech Hub]] · [[Simulador Navecon]]
- Mapa: [[Frontend]]
