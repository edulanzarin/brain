---
tags: [tipo/atomica, camada/padrao, infra, windows, armadilha]
criado: 2026-09-25
---

# No Git Bash, caminho Unix em argumento vira caminho do Windows

> O Git Bash (MSYS) reescreve todo argumento que parece caminho Unix antes de chamar o programa. `docker exec banco pg_restore /tmp/x.dump` chega ao Docker como `C:/Users/.../Temp/x.dump`, e o container responde que o arquivo não existe.

## O problema

Num ensaio de dump e restore pelo Git Bash:

```bash
docker exec navex-db pg_restore -U navex -d navex /tmp/nexo.dump
# pg_restore: error: could not open input file "C:/Users/edula/AppData/Local/Temp/nexo.dump"
```

O mesmo comando funciona no Linux, no PowerShell e dentro de `sh -c '...'`. Três vizinhos na mesma sessão não foram tocados, o que confunde o diagnóstico:
- `docker cp navex-db:/tmp/nexo.dump .`: o prefixo `nome:` tira o argumento do padrão de caminho.
- `sh -c 'pg_dump ... -f /tmp/nexo.dump'`: o argumento inteiro não começa com `/`.
- `docker exec navex-db rm /tmp/nexo.dump`: este foi convertido e falhou do mesmo jeito.

## A solução

Desligar a conversão na sessão:

```bash
export MSYS_NO_PATHCONV=1
```

Ou passar o caminho com barra dupla no início (`//tmp/nexo.dump`), que o MSYS não reescreve e o Linux lê igual.

## O que mais vale lembrar

- O sintoma entrega a causa: a mensagem de erro traz um caminho do Windows que o comando não tinha. Quando o erro mostra um caminho que você não escreveu, suspeite do shell antes do programa.
- Vale para qualquer programa que receba caminho de outro sistema de arquivos: `docker exec`, `ssh host comando`, `kubectl exec`. Roteiro que vai rodar no Git Bash de alguém leva o `export` no começo.
- O mesmo shell que ajuda (`/c/Dev` vira `C:\Dev` para programa Windows) atrapalha quando o destino não é o Windows.

## Conexões
- Princípio: (folha isolada: nenhum princípio da Base cobre; é compatibilidade de shell)
- Irmã: [[No Windows o npm roda script pelo cmd.exe, e a porta padrão do script dev chega literal]] · [[No Windows, duas coisas escutam a mesma porta e o cliente fala com a errada]]
- Visto em: [[NaveX]] (ensaio da troca do nexo2)
- Mapa: [[Infra]]
