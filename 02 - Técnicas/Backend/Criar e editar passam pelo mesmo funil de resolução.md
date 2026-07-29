---
tags: [tipo/atomica, camada/padrao, dev/backend, armadilha]
criado: 2026-07-29
---

# Criar e editar passam pelo mesmo funil de resolução

> A regra que decide algo derivado de um registro — o dono, a validação, o
> vínculo por chave natural — tem que rodar em TODO caminho de escrita. Quando só
> o cadastro a aplica e a edição tem a sua própria versão (ou nenhuma), a edição
> vira porta dos fundos: aceita, em silêncio, estados que o cadastro recusaria.

## A armadilha

O cadastro costuma nascer caprichado: resolve o dono, valida, cria dependência.
A edição é escrita depois, "só pra ajustar campos", e recebe uma cópia da regra —
ou passa direto. As duas divergem, e o bug aparece exatamente pela rota que
parecia inofensiva.

No Cofre Digital, o `POST` de certificado resolvia a empresa dona pelo CNPJ
(`resolveCompany`: e-CNPJ sem empresa entra no cofre do dono, criando na primeira
vez). O `PUT` **não** tinha isso — usava o `companyId` que veio do formulário e
pronto. Consequências, todas silenciosas:

- Trocar por um `.pfx` de outro CNPJ deixava o certificado **pendurado na empresa
  original** (o titular/documento mudavam, o dono não).
- O "Automático pelo CNPJ" na edição não religava nada — zerava o vínculo e o
  registro virava **órfão**.
- Nada barrava um e-CNPJ preso numa empresa de CNPJ diferente.

## O padrão

Uma função só de resolução/validação, chamada **igual** pelos dois caminhos:

```ts
// certificate-api.ts — uma regra, usada por POST e PUT
export async function resolveCertCompany(data): Promise<CompanyResolution> {
  // empresa escolhida à mão: é ela, MAS e-CNPJ tem que bater com o CNPJ dela
  // sem empresa + e-CNPJ: cofre do dono do CNPJ (cria na 1a vez)
  // e-CPF: sem checagem de CNPJ
}
```

O ganho não é só menos código: é que a edição **não consegue** aceitar o que o
cadastro recusa, porque é a mesma peça decidindo. O vínculo passa a seguir a
**chave natural do dado** (o CNPJ), resolvida no servidor, não o estado que
sobrou no formulário. Invariante garantido pela estrutura, não pela lembrança de
cada rota — [[Um invariante se garante na estrutura, não no processo]].

## O que mais vale lembrar

É o lado da escrita do mesmo raciocínio de [[Filtro transversal só é honesto se
todo o funil o honra]]: lá toda CONSULTA precisa honrar o filtro compartilhado;
aqui toda ESCRITA precisa passar pelo mesmo resolvedor. Query própria e rota de
edição própria são o mesmo tipo de buraco — o que escapa do funil escapa da
regra. Ao adicionar uma regra ao cadastro, a pergunta seguinte é sempre "a edição
também passa por aqui?".

## Conexões
- Princípio: [[Um invariante se garante na estrutura, não no processo]]
- Irmã: [[Filtro transversal só é honesto se todo o funil o honra]]
- Irmã: [[Trocar o arquivo repede a senha e relê os dados]]
- Visto em: [[Cofre Digital]]
- Mapa: [[Backend]]
