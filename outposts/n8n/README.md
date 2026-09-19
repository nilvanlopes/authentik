# Authentik Outpost for n8n

Outpost manual do Authentik para proteger o editor do n8n via Traefik
`forwardAuth`.

## Objetos persistentes

O provisionador da raiz cria ou valida:

- grupo `n8n`, contendo somente o usuário `pyu`;
- Application `n8n`;
- Proxy Provider `n8n-forward-auth` em modo `Forward auth (single application)`;
- Outpost manual `n8n`, do tipo `Proxy`, associado ao provider.

O provider libera apenas os caminhos necessários para automações:

```text
^/webhook(?:/|$)
^/webhook-test(?:/|$)
^/webhook-waiting(?:/|$)
^/mcp(?:/|$)
^/mcp-server(?:/|$)
```

Essas exceções não removem CrowdSec nem a autenticação nativa configurada em
cada webhook ou no MCP do n8n.

## Provisionamento

Na raiz do repositório, usando um token administrativo local do Authentik:

```bash
read -rsp "Authentik API token: " AUTHENTIK_API_TOKEN; echo
export AUTHENTIK_API_TOKEN
make provision-authentik-n8n
unset AUTHENTIK_API_TOKEN
```

O token administrativo não é salvo, impresso ou commitado. O provisionador é
idempotente e aborta se encontrar um grupo `n8n` com membros além de `pyu` ou
objetos existentes com configuração incompatível.

Depois, copie `.env.example` para `.env` e coloque o token gerado no Authentik:

```bash
cp authentik/outposts/n8n/.env.example authentik/outposts/n8n/.env
chmod 600 authentik/outposts/n8n/.env
```

## Deploy

```bash
make deploy-authentik-outpost-n8n
make deploy-traefik
make deploy-n8n
```

O serviço deve ficar como `authentik-outpost-n8n_proxy`.

## Verificação

- `https://n8n.nilvanlopes.com/outpost.goauthentik.io/ping` responde sem erro;
- uma sessão sem autenticação é redirecionada para o Authentik;
- somente o usuário `pyu`, membro do grupo `n8n`, acessa o editor;
- webhooks e MCP não exigem sessão de navegador, mas rejeitam credenciais
  nativas inválidas;
- o serviço do outpost fica em `1/1`.
