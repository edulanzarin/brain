---
tags: [tipo/atomica, camada/padrao, dev/frontend]
criado: 2026-07-20
---

# Cache do React Query não é lugar de estado de interface

> `setQueryData` numa chave inventada parece um store global de graça. Mas
> chave sem observador é coletada pelo `gcTime`, e o estado some sozinho
> depois de alguns minutos.

## O quê

O truque tentador para fazer estado sobreviver à navegação:

```ts
queryClient.setQueryData(["meu-estado"], valor);          // guarda
const [x] = useState(() => queryClient.getQueryData(...)); // restaura
```

Funciona no teste rápido e falha no uso real. O React Query é um cache de
**resposta de servidor**: toda query sem observador entra em coleta e é
removida depois do `gcTime` (5 min por padrão). Como esse `getQueryData` no
inicializador não cria observador, ninguém segura a chave — trabalhar 5
minutos numa aba e voltar encontra o estado limpo.

Para estado de interface, um `Map` de módulo resolve melhor: não expira, não
tem status/stale/refetch que não se usa, e o descarte fica explícito onde deve
estar (ver [[Estado de tela pertence à seção, não à página]]).

## Por que importa

O bug é traiçoeiro porque **depende do tempo**: passa em todo teste manual
rápido e só aparece com o usuário que foi fazer outra coisa e voltou. O
sintoma ("perdi o que eu tinha carregado") também não aponta para o cache.

Regra que sobra: no React Query vai o que veio do servidor e pode ser
rebuscado. O que só existe no cliente — recorte de tela, arquivo já
processado, escolha temporária — mora em outro lugar.

## Conexões
- Princípio: [[Estado compartilhável mora na URL]]
- Ver também: [[Estado de tela pertence à seção, não à página]]
- Visto em: [[Questor BI]], na Conciliação (extrato lido guardado no cache)
- Mapa: [[Frontend]]
