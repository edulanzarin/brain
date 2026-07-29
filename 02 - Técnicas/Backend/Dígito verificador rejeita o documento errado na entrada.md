---
tags: [tipo/atomica, camada/padrao, dev/backend]
criado: 2026-07-29
---

# Dígito verificador rejeita o documento errado na entrada

> Um identificador que carrega o próprio dígito verificador (CPF, CNPJ, e tantos
> outros) deixa o funil de escrita **recusar o erro de digitação** de forma
> determinística — antes que um número trocado crie ou vincule, em silêncio, a
> entidade errada lá na frente.

## O problema

Validar só o **tamanho** ("14 dígitos = CNPJ") não basta. O erro de digitação
mais comum mantém o tamanho certo e vira outro número — que aponta pra uma
entidade que não existe. E quando o sistema resolve o dono pela chave natural
("Automático pelo CNPJ": sem empresa, o e-CNPJ entra no cofre do dono, criando na
primeira vez), esse CNPJ trocado **cria uma empresa errada** sem reclamar. O erro
não aparece na hora: vira um registro órfão ou pendurado no dono errado, semanas
depois.

## A solução

Conferir o dígito verificador no **mesmo funil** que cadastro e edição
compartilham ([[Criar e editar passam pelo mesmo funil de resolução]]), e recusar
com mensagem específica — não um genérico "dados inválidos".

```ts
export function isValidCnpj(doc: string): boolean {
  const c = doc.replace(/\D/g, "");
  if (c.length !== 14 || /^(\d)\1{13}$/.test(c)) return false; // barra 000..., 111...
  const w1 = [5, 4, 3, 2, 9, 8, 7, 6, 5, 4, 3, 2];
  const w2 = [6, 5, 4, 3, 2, 9, 8, 7, 6, 5, 4, 3, 2];
  const check = (w: number[]) => {
    const s = w.reduce((a, wi, i) => a + Number(c[i]) * wi, 0);
    const r = s % 11;
    return r < 2 ? 0 : 11 - r;
  };
  return check(w1) === Number(c[12]) && check(w2) === Number(c[13]);
}
// CPF: pesos decrescentes (10..2 e 11..2), resto*10 % 11, 10 vira 0.
```

O documento vindo de um arquivo assinado (um `.pfx` ICP-Brasil) sempre passa — o
erro só entra pela digitação à mão. Como o funil é um só, ele pega os dois
caminhos sem precisar distinguir a origem. Vale também gravar o documento
**canônico** (sempre formatado) no mesmo ponto: o anti-duplicata passa a comparar
maçã com maçã.

## O que mais vale lembrar

É o gêmeo de entrada do [[Criar e editar passam pelo mesmo funil de resolução]]:
o funil decide o **dono**; o dígito verificador decide se a **identidade é real**
antes de chegar lá. Sem ele, o funil resolve com capricho um dono que nunca
deveria ter existido.

Bloquear, não avisar: um número que falha no dígito é erro, não escolha. E quando
a validação estrita passa a recusar a **própria fixture de teste** (documento
fake que nunca conferiu), a saída é regenerar a fixture com um documento válido —
não afrouxar a verificação. A fixture existe pra exercitar o invariante, não pra
furá-lo — [[Semear teste cria linha nova, não muta linha real]].

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[Criar e editar passam pelo mesmo funil de resolução]]
- Visto em: [[Cofre Digital]]
- Mapa: [[Backend]]
