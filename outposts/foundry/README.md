# Authentik Outpost for Foundry

Outpost manual do Authentik para proteger o Foundry via `forwardAuth`.

## Arquivos

- [`docker-compose.yml`](./docker-compose.yml): stack do outpost em Swarm
- [`.env.example`](./.env.example): variaveis necessarias para o token e URL do Authentik

## Como configurar no Authentik

1. Crie uma `Application` para o Foundry.
2. Crie um `Proxy Provider` em modo `Forward auth (single application)`.
3. Crie um `Outpost` do tipo `Proxy`.
4. Escolha deployment manual.
5. Copie o token gerado para [`.env`](./.env).

Use o host publicado pelo Traefik para o Foundry:

```text
https://foundry.nilvanlopes.com
```

## Deploy

No contexto da raiz deste repositorio:

```bash
cp authentik/outposts/foundry/.env.example authentik/outposts/foundry/.env
make deploy-authentik-outpost-foundry
```

## Verificacao

- o servico deve ficar como `authentik-outpost-foundry_proxy`
- a rota `https://foundry.nilvanlopes.com/outpost.goauthentik.io/ping` deve responder sem erro
- o middleware `authentik-foundry-middleware@file` no Traefik deve conseguir acessar `http://authentik-outpost-foundry_proxy:9000`
