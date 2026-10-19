# Outpost do portal Fichas de RPG

Os arquivos de configuração ficam em `authentik/outposts/fichasrpg/`.
O outpost é publicado na stack `authentik-outpost-fichasrpg`.

Outpost manual dedicado a `https://fichasrpg.nilvanlopes.com`, com proxy
provider em modo `forward_single`. O acesso exige participação no grupo
`fichasrpg-users`. O provisionador copia para ele os membros existentes do
grupo `fichamutantes`, incluindo `pyu`; contas que só pertencem a
`foundry-users` não são adicionadas.

Na raiz do repositório Docker:

```bash
make provision-authentik-fichasrpg
make deploy-authentik-outpost-fichasrpg
make deploy-fichasrpg
make activate-fichasrpg-routes
```

O provisionador usa o shell administrativo do container Authentik local,
cria os objetos de forma idempotente, migra o acesso das contas atuais e salva o token do outpost em
`.env`, ignorado pelo Git e com permissão `600`. Objetos existentes com
configuração incompatível causam erro em vez de serem sobrescritos.
O provider recebe os defaults OAuth do próprio Authentik, incluindo callbacks
estritos no domínio da ficha, scopes do proxy e cliente confidencial.
O provisionador também remove da aplicação os vínculos antigos com
`authentik Admins` e `foundry-users`, criados no primeiro deploy. Esses grupos
e as permissões do Foundry continuam sendo administrados na própria aplicação.

O arquivo `traefik/infra/traefik/templates/fichasrpg-routes.yml` contém as novas
rotas. O alvo `make activate-fichasrpg-routes` copia essa configuração para a
pasta monitorada pelo file provider do Traefik. O cadastro por convite é
público; o portal, as fichas e as APIs privadas exigem sessão no grupo
`fichasrpg-users`. A aplicação e os endpoints de outpost passam pelo CrowdSec.

Depois de conferir login, criação de fichas e cadastro no novo domínio, ative
os redirects antigos com `make activate-fichasrpg-redirects`.

Para validar o deploy, confira as réplicas de
`authentik-outpost-fichasrpg_proxy`, o HTTP 204 em
`/outpost.goauthentik.io/ping` e o redirecionamento ao Authentik ao abrir o portal
sem uma sessão autenticada.
Siga também o redirecionamento até a tela de login: somente conferir o primeiro
HTTP 302 não detecta erros na URL de retorno OAuth.
