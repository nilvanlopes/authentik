# Authentik Outpost for Traefik Dashboard

Outpost manual do Authentik para proteger o dashboard do Traefik via `forwardAuth`.

## Arquivos

- [`docker-compose.yml`](./docker-compose.yml): stack do outpost em Swarm
- [`.env.example`](./.env.example): variaveis necessarias para o token e URL do Authentik

## Como configurar no Authentik

1. Crie uma `Application` para o dashboard do Traefik.
2. Crie um `Proxy Provider` em modo `Forward auth (single application)`.
3. Crie um `Outpost` do tipo `Proxy`.
4. Escolha deployment manual.
5. Copie o token gerado para [`.env`](./.env).

Use o mesmo host publicado pelo Traefik para o dashboard, por exemplo:

```text
https://traefik.nilvanlopes.com
```

## Variaveis

```env
AUTHENTIK_HOST=https://authentik.seu-dominio.com
AUTHENTIK_TOKEN=token-gerado-no-authentik
AUTHENTIK_INSECURE=false
# AUTHENTIK_HOST_BROWSER=https://authentik.seu-dominio.com
```

`AUTHENTIK_HOST_BROWSER` so e necessario quando a URL usada pelo outpost para falar com o core do Authentik difere da URL publica exibida ao navegador.

## Deploy

No contexto da raiz deste repositorio:

```bash
cp authentik/outposts/traefik/.env.example authentik/outposts/traefik/.env
make deploy-authentik-outpost-traefik
```

## Verificacao

- o servico deve ficar como `authentik-outpost-traefik_proxy`
- a rota `https://traefik.nilvanlopes.com/outpost.goauthentik.io/ping` deve responder sem erro
- o middleware `authentik-traefik-middleware@file` no Traefik deve conseguir acessar `http://authentik-outpost-traefik_proxy:9000`
