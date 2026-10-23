# Outpost do site pessoal

Outpost dedicado `meu-site-outpost`, vinculado somente ao provider
`meu-site-forward-auth` e à aplicação `meu-site`, para
`https://meusite.nilvanlopes.com`.

O grupo `meu-site` contém somente `pyu`; a política também exige que a conta
esteja ativa e tenha esse nome. O provisionamento fica em
`scripts/setup-meu-site.py` e `scripts/meu-site-authentik.py`, na raiz do
repositório de infraestrutura.

O token é reutilizado do Authentik e salvo localmente em `.env`, com permissão
`0600` e ignorado pelo Git. A partir da raiz da infraestrutura:

```bash
make deploy-authentik-outpost-meu-site
docker stack services authentik-outpost-meu-site
```

O serviço deve ficar `1/1` e saudável; o ping público em
`/outpost.goauthentik.io/ping` retorna HTTP 204. O provider registra o callback
completo, incluindo `?X-authentik-auth-callback=true`.

Use uma mensagem simples em `${AUTHENTIK_TOKEN:?authentik token required}`:
o interpolador de `docker stack` neste host interpretou incorretamente uma
mensagem contendo o caminho `scripts/setup-meu-site.py`, substituindo o token
por parte do texto da mensagem.

Os limites são 0,25 CPU e 128 MiB, com raiz somente leitura e tmpfs em `/tmp`.
As sessões temporárias podem ser perdidas ao reiniciar o serviço.
