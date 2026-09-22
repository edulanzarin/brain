---
tags: [tipo/atomica, camada/padrao, infra, docker, armadilha]
criado: 2026-09-22
---

# O túnel publica alcançando o container pelo nome, sem abrir porta

> No servidor da Navecon (`ts05`) não há nginx, não há Caddy e **nada escuta a
> 80 ou a 443**. Quem traz o domínio é um **Cloudflare Tunnel**: um container
> `cloudflared` que abre conexão de saída e roteia `hostname → container`. Todo
> compose que eu escrevia com Caddy próprio resolvia um problema que aquele host
> não tem.

## O arranjo

Um `cloudflared` só, com um `config.yml` local:

```yaml
tunnel: <id>
credentials-file: /etc/cloudflared/<id>.json
ingress:
  - hostname: simulador.navecon.net.br
    service: http://simulador-navecon-app:3000
  - service: http_status:404      # coringa, sempre por último
```

O `service` aponta para o **nome do container**, resolvido pelo DNS interno do
Docker. Por isso o app precisa estar na **rede do cloudflared**, e é só isso que
"publicar" significa aqui.

## Pôr um projeto no ar, em três passos

1. **Entrar na rede do túnel**, por um `docker-compose.override.yml` que existe
   só no servidor:

   ```yaml
   services:
     app:
       networks: [simulador-navecon-net, bolao-net]
   networks:
     bolao-net:
       external: true
       name: bolao-navepro_bolao-net
   ```

   As duas redes são obrigatórias. Declarar `networks:` num serviço o tira da
   rede default do projeto, e sem a própria o app perde o banco — a armadilha já
   descrita em
   [[Dois projetos no mesmo host se falam por rede externa compartilhada]].

2. **Acrescentar a regra de ingress** antes do coringa `http_status:404`, e
   validar antes de aplicar:

   ```bash
   docker run --rm -v ~/…/cloudflared:/etc/cloudflared:ro cloudflare/cloudflared \
     --config /etc/cloudflared/config.yml tunnel ingress validate
   ```

   `--config` é flag **global**, vem antes de `tunnel`. Depois do subcomando ele
   é ignorado e o comando só imprime a ajuda, que parece erro de sintaxe e não é.

3. **Criar o CNAME** `sub` → `<id>.cfargotunnel.com`, proxied. Isso **não dá
   para fazer do servidor** quando só existe o JSON de credencial do túnel:
   `cloudflared tunnel route dns` exige o `cert.pem` de conta. Sem o CNAME a
   regra de ingress está certa e o domínio simplesmente não resolve.

## SIGHUP não recarrega o cloudflared, mata

A documentação de muitos daemons promete recarga com `SIGHUP`, e o reflexo é
`docker kill -s HUP`. No `cloudflared` isso **encerra o processo** (`Exited (2)`),
e com `restart: unless-stopped` o Docker não levanta de volta, porque `kill`
conta como parada deliberada.

Aplicar mudança de ingress é `docker restart`, e custa os ~30 s de reconexão do
túnel. Planeje a janela: aqui deu 36 s fora do ar em dois domínios que estavam
em produção.

## A fragilidade que fica

O `cloudflared` mora **dentro do compose de outro projeto** (o do bolão). Quem
rodar `docker compose down` naquele projeto derruba o túnel e leva junto todos
os domínios, inclusive os de projetos que nada têm a ver com ele.

O túnel é infraestrutura compartilhada e devia ter compose próprio, como a rede
`navecon-ponte` de [[Infra]]. Enquanto não tiver, vale um aviso no README de
cada projeto que depende dele.

## Conexões
- Irmã: [[Dois projetos no mesmo host se falam por rede externa compartilhada]] ·
  [[No Windows, duas coisas escutam a mesma porta e o cliente fala com a errada]]
- Princípio: [[Config declarada envelhece; quem diz a regra é o comportamento observado]]
- Visto em: [[Simulador Navecon]] · [[Evento Navecon]]
- Mapa: [[Infra]]
