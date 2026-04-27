# Authentik Outpost for Portainer

Outpost manual do Authentik para proteger o Portainer via `forwardAuth`.

## Arquivos

- [`docker-compose.yml`](./docker-compose.yml): stack do outpost em Swarm
- [`.env.example`](./.env.example): variaveis necessarias para o token e URL do Authentik

## Como configurar no Authentik

1. Crie uma `Application` para o Portainer.
2. Crie um `Proxy Provider` em modo `Forward auth (single application)`.
3. Crie um `Outpost` do tipo `Proxy`.
4. Escolha deployment manual.
5. Copie o token gerado para [`.env`](./.env).

Use o host publicado pelo Traefik para o Portainer, por exemplo:

```text
https://portainer.nilvanlopes.com
```

## Variaveis

```env
AUTHENTIK_HOST=https://authentik.seu-dominio.com
AUTHENTIK_TOKEN=token-gerado-no-authentik
AUTHENTIK_INSECURE=false
AUTHENTIK_HOST_BROWSER=https://authentik.seu-dominio.com
```

## Deploy

No contexto da raiz deste repositorio:

```bash
cp authentik/outposts/portainer/.env.example authentik/outposts/portainer/.env
make deploy-authentik-outpost-portainer
```

## Verificacao

- o servico deve ficar como `authentik-outpost-portainer_proxy`
- a rota `https://portainer.nilvanlopes.com/outpost.goauthentik.io/ping` deve responder sem erro
- o middleware `authentik-portainer-middleware@file` no Traefik deve conseguir acessar `http://authentik-outpost-portainer_proxy:9000`
