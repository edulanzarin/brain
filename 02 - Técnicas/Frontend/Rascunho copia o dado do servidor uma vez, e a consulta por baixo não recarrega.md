---
tags: [tipo/atomica, camada/padrao, dev/frontend, armadilha]
criado: 2026-09-25
---

# Rascunho copia o dado do servidor uma vez, e a consulta por baixo não recarrega

> Janela de edição que lê o registro por React Query e copia para um
> `useState` tem dois jeitos de mentir sem erro nenhum. Com cache, ela copia a
> versão velha e ignora a nova que chega logo depois. Sincronizando o rascunho
> "sempre que o dado mudar", a volta à aba do navegador recarrega o registro e
> apaga o que a pessoa digitou.

## O caso

No [[NaveX]], a janela de um grupo de empresa busca o grupo gravado (nome, modo,
empresas marcadas) e a pessoa edita uma cópia. Os padrões do React Query jogam
contra:

- **`gcTime` de 5 minutos**: fechar e reabrir o mesmo grupo depois de salvar
  entrega primeiro o dado do cache (anterior ao salvamento) e depois o novo. Se a
  cópia é feita no primeiro dado, a janela abre com o grupo de antes.
- **`refetchOnWindowFocus`**: a pessoa vai conferir um CNPJ em outra aba, volta,
  a consulta recarrega. Se um `useEffect([data])` recopia o rascunho, a edição
  some.

As duas saídas óbvias se contradizem: recopiar a cada dado novo resolve a
primeira e causa a segunda; copiar só uma vez resolve a segunda e mantém a
primeira.

## A regra

O registro é lido **fresco a cada abertura** e **não recarrega enquanto a
janela estiver aberta**. A cópia acontece **uma vez por abertura**:

```ts
const detalhe = useQuery({
  queryKey: ["grupo", id],
  queryFn: () => buscar(id),
  enabled: id != null,
  gcTime: 0,                    // fechou, sumiu: a próxima abertura busca de novo
  staleTime: Infinity,          // aberta, não recarrega sozinha
  refetchOnWindowFocus: false,
});

// Ajuste no render, chaveado pela origem: copia quando a abertura muda.
const origem = aberta ? (detalhe.data ? `g${detalhe.data.id}` : null) : null;
const [copiadoDe, setCopiadoDe] = useState<string | null>(null);
const [rascunho, setRascunho] = useState<Rascunho | null>(null);
if (origem !== copiadoDe) {
  setCopiadoDe(origem);
  setRascunho(origem ? copiar(detalhe.data) : null);
}
```

Guarde também a cópia original: é contra ela que "há o que salvar" se decide
(Salvar só acende com mudança, e com mudança o clique fora não fecha).

A alternativa de remontar pela `key` (ver a irmã abaixo) também funciona quando o
componente que guarda o rascunho pode nascer depois do dado. Aqui não podia: o
rodapé com o botão de salvar é da janela, que já está aberta mostrando o
esqueleto enquanto o dado chega.

## Conexões
- Irmã: [[Ajustar estado no render é legítimo, empurrar rota não é]] (o mecanismo da
  cópia) · [[Trocar de sujeito na mesma rota não remonta, e o estado do anterior fica]]
  · [[Cache do React Query não é lugar de estado de interface]] (o outro sentido:
  estado de interface guardado no cache)
- Visto em: [[NaveX]]
- Mapa: [[Frontend]]
