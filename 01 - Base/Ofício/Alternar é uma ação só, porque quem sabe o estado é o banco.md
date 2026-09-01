---
tags: [tipo/atomica, camada/principio, dev, dev/backend]
criado: 2026-09-01
---

# Alternar é uma ação só, porque quem sabe o estado é o banco

> Guardar e tirar dos favoritos, seguir e deixar de seguir, marcar e desmarcar:
> parecem duas operações opostas e por isso viram duas funções. São uma só, e
> quem decide qual dos dois sentidos vai acontecer não é quem clicou — é o que
> estava gravado no instante da escrita.

## A regra

**Uma ação `alternar(alvo)`, e o sentido se decide dentro dela, contra o banco.**
Não `guardar()` e `tirar()`, escolhidos pela tela conforme o coração está cheio
ou vazio.

E a decisão sai de uma escrita, não de uma leitura seguida de escrita:

```ts
// o apagar JÁ é o teste: quantas linhas caíram responde se existia
const { count } = await db.favorito.deleteMany({ where: { usuarioId, alvoId } });
if (count > 0) return { guardado: false };

await db.favorito.create({ data: { usuarioId, alvoId } });
return { guardado: true };
```

E a resposta devolve o estado resultante, para a tela adotá-lo em vez de manter
o palpite dela.

## Por que

Duas razões, e a segunda é a que morde.

**A tela não é dona do estado.** Com `guardar()` e `tirar()` separadas, quem
escolhe qual chamar é o que o navegador acha que está gravado. Duas abas abertas
no mesmo perfil, ou uma volta pelo histórico, e esse palpite já nasce errado: a
aba antiga chama `guardar()` no que já está guardado, ou `tirar()` no que já
saiu. O resultado é um erro que só acontece com quem tem duas abas — o defeito
mais caro de reproduzir que existe.

**Perguntar antes de escrever abre uma janela.** `existe?` e depois `apaga` são
dois momentos, e entre eles cabe o segundo clique: os dois leem "não existe" e
os dois concluem que precisam criar. Apagar primeiro e olhar o número de linhas
que caíram é a mesma decisão feita numa escrita atômica, sem janela nenhuma.

A criação simultânea ainda pode bater na chave única — e é ela que resolve, não
mais uma leitura: quem colidiu recebe o mesmo estado final que pediu.

## Na prática

- **Idempotência é o teste.** Chamar duas vezes com o mesmo clique tem que
  acabar no mesmo lugar de chamar uma. Se não acaba, o sentido está vindo de
  fora.
- **A tela pode virar o coração antes da resposta** — é um clique dado enquanto
  se rola a lista, e esperar o ida e volta parece botão que não pegou. Mas ela
  adota o que voltou, e desfaz se veio erro.
- **A regra vale para o par que compartilha uma linha só.** Papel de mídia
  (retrato, capa), marca de leitura, curtida: em todos, ligar é o inverso do que
  está gravado.
- Não vale quando os dois sentidos têm regras diferentes — apagar uma avaliação
  não pede o mesmo que escrevê-la. Aí são duas ações porque são duas regras, e
  não porque são dois botões.

## Conexões
- Depende de: [[Estado mutável se lê da fonte no uso, não de cópia guardada]] — o
  caso extremo dela: nem para decidir o que fazer a cópia da tela serve.
- Irmã: [[A regra mora fora da porta que a chama]]
- Irmã: [[Um invariante se garante na estrutura, não no processo]] — a chave
  única é o que sobra segurando a corrida.
- Visto em: [[Privello]]
- Mapa: [[Base]]
