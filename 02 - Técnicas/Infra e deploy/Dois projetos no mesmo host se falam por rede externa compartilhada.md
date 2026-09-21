---
tags: [tipo/atomica, camada/padrao, infra, dev/backend]
criado: 2026-09-21
---

# Dois projetos no mesmo host se falam por rede externa compartilhada

> Cada compose cria a sua rede (`<slug>-net`) e containers de stacks diferentes não se enxergam, nem por nome nem por IP interno. Rodar no mesmo PC não basta. A saída é uma terceira rede, criada fora dos dois, em que os dois entram — não publicar porta a mais.

## O problema

O chassi do [[Infra]] dá a cada projeto a sua rede. É o certo: o projeto sobe e cai
inteiro, sem depender de vizinho. O preço aparece no dia em que um sistema precisa de
um dado que é do outro — no cofre e no painel, o certificado de um cliente.

Duas saídas erradas se apresentam primeiro, e as duas custam mais do que parecem:

**Publicar o banco na rede.** Trocar `127.0.0.1:5004:5432` por `5004:5432` resolve em
dez segundos e abre o banco inteiro para qualquer máquina da rede — com a senha que o
compose tem por padrão, sem TLS, sem trilha de quem leu o quê. E entrega o schema como
contrato: a partir daí, renomear uma coluna quebra o outro sistema. Contraria
[[Porta interna é constante, porta externa é configuração]], que já diz a quem cada
porta é publicada e por quê.

**Falar pelo IP do host.** `http://192.168.5.68:4004` funciona hoje e prende o projeto
a um IP que muda quando a máquina muda. O tráfego ainda sai pela interface do host e
volta, então o que passa ali — senha, arquivo — trafega na rede do escritório para ir
de um container ao container ao lado.

## A receita

Uma rede criada uma vez, fora de qualquer projeto:

```bash
docker network create navecon-ponte
```

Cada compose que participa declara essa rede como **externa** e lista o serviço nela:

```yaml
services:
  app:
    networks:
      - default      # a rede do próprio projeto — ver a armadilha abaixo
      - ponte
networks:
  default:
    name: cofre-digital-net
  ponte:
    external: true
    name: navecon-ponte
```

Feito isso, um chama o outro por **nome de serviço e porta interna**
(`http://cofre-digital-app:3000`), igual ao que já se faz dentro do compose. Nenhuma
porta nova publicada, o tráfego não sai do Docker, e cada stack continua subindo e
caindo sozinha — a ponte é ponto de encontro, não dependência de ordem.

### A armadilha do `default`

No Compose, **assim que um serviço declara `networks:`, ele sai da rede default**. Um
serviço que antes não declarava nada e ganha só `ponte` perde o próprio banco: o
`app-db:5432` deixa de resolver e o app sobe sem banco, com erro de conexão que não
menciona rede nenhuma. Listar `default` junto é obrigatório, não estilo.

## O que atravessa a ponte é API, não SQL

A ponte é transporte; ela não decide o contrato. O dado continua saindo pelo app que é
dono dele, por rota HTTP, com a permissão e o registro que aquele sistema já tem. Banco
compartilhado não sabe quem leu; rota sabe.

## Conexões
- Princípio: [[Porta interna é constante, porta externa é configuração]]
- Depende de: [[O nome do projeto governa o nome dos recursos]]
- Ver também: [[Fornecedor externo entra pelo contrato do app, não o app pelo dele]]
- Visto em: [[Cofre Digital]]
- Mapa: [[Infra]]
