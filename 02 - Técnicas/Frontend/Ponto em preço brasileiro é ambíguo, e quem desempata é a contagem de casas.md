---
tags: [tipo/atomica, camada/padrao, armadilha, dev/frontend]
criado: 2026-08-30
---

# Ponto em preço brasileiro é ambíguo, e quem desempata é a contagem de casas

> "1.200" são mil e duzentos. "250.50" são duzentos e cinquenta e cinquenta.
> O mesmo ponto, dois papéis — e o parser ingênuo transforma R$ 1.200 em R$ 1,20
> sem dar erro nenhum.

## O problema

O campo de preço aceita o que a pessoa digitar, e ela digita das três formas
que aprendeu na vida: `1200`, `1.200` e `1.200,00`. A regra "havendo vírgula, o
ponto é milhar; não havendo, o ponto é decimal" resolve duas e erra a do meio:

```
"1.200"  → 1.2  → 120 centavos   ← R$ 1,20 no banco
```

O defeito não levanta exceção, não falha validação e não aparece em teste de
tipo. Ele publica um preço errado, e quem escreveu não confere de novo porque
digitou certo.

## A solução

Dinheiro não tem três decimais. Então três dígitos depois do último ponto só
podem ser separador de milhar, e mais de um ponto também:

```ts
if (cru.includes(",")) {
    normal = cru.replace(/\./g, "").replace(",", ".");   // vírgula manda
} else {
    const pontos = cru.split(".").length - 1;
    const casas = cru.length - cru.lastIndexOf(".") - 1;
    const ehMilhar = pontos > 1 || (pontos === 1 && casas === 3);
    normal = ehMilhar ? cru.replace(/\./g, "") : cru;
}
```

Cobre `250`, `1.200`, `1.200,00`, `1.200.000`, `250,50`, `250.50` e `250.5`. E
a conta de volta — número para texto — já tinha que estar num formatador só,
pelo mesmo motivo: `toFixed` devolve ponto, e em português ponto é milhar.

## O que mais vale lembrar

Só apareceu **enviando o formulário de verdade** e olhando a linha no banco. O
typecheck passa, a validação passa, a tela mostra o que foi digitado. A conta
errada mora no meio, onde ninguém olha.

## Conexões
- Princípio: [[Casar dado do mundo real é por classe de equivalência, não por igualdade]] —
  o mesmo valor chega em várias grafias corretas, e comparar a string em vez da
  classe é onde a leitura silenciosamente muda o dado.
- Irmã: [[CSV que abre no Excel pt-BR usa ponto e vírgula, BOM e vírgula decimal]]
- Visto em: [[Privello]]
- Mapa: [[Frontend]]
