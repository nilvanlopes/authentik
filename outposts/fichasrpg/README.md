# Outpost do FichasRPG

Outpost dedicado a `https://fichasrpg.nilvanlopes.com`, publicado na stack
`authentik-outpost-fichasrpg`. O provider `fichamutantes-forward-auth` usa
`forward_single`; a aplicação `fichamutantes` exige o grupo `fichasrpg-users`.
Esses identificadores persistentes são os da aplicação atual.

A aplicação, o provider, o grupo e o outpost já existem no Authentik deste
ambiente. Em um novo ambiente, configure esses objetos pelo painel e salve
o token do outpost em `.env`, baseado em `.env.example`.

Na raiz do repositório Docker:

```bash
make deploy-authentik-outpost-fichasrpg
make deploy-fichasrpg
```

As configurações de rota existem somente em
`traefik/infra/traefik/dynamic/fichasrpg.yml`. O Traefik acompanha essa pasta diretamente; não há
comando para copiar ou ativar templates. O cadastro público concede somente
`fichasrpg-users`. A API valida diretamente a sessão nesse grupo consultando o outpost, sem
`forwardAuth` duplicado no Traefik. O Traefik encaminha o portal, os endpoints
de sessão e o domínio de login em três rotas.

O login é servido pelo Authentik existente em
`https://login-fichasrpg.nilvanlopes.com`, definido em `AUTHENTIK_HOST_BROWSER`.
O host interno é `http://server:9000`. Os demais outposts mantêm seus endereços.
O cliente Google do FichasRPG autoriza o callback
`https://login-fichasrpg.nilvanlopes.com/source/oauth/callback/google-fichasrpg/`.

Para validar, confira a réplica do outpost, HTTP 204 em
`/outpost.goauthentik.io/ping`, login completo e logout. Siga todos os redirects
até a tela de login; apenas o primeiro HTTP 302 não valida o retorno OAuth.
