# Authentik Docker Compose Setup

## Versão em uso

Servidor, worker e os outposts usam as imagens oficiais **2026.8.4**,
fixadas por digest nos arquivos Compose. A migração de 2026.2.7 passou por
2026.5.7 antes de chegar a 2026.8.4. Em futuras atualizações, mantenha todos
esses componentes na mesma versão e siga a sequência de versões principais
indicada na [documentação oficial](https://docs.goauthentik.io/install-config/upgrade/).

Os outposts preservam sessões em volumes individuais montados em
`/var/lib/authentik-outpost-session`. O servidor e os outposts confiam nos
cabeçalhos encaminhados pela rede `traefik-public` (`10.0.1.0/24`). O servidor
também confia na rede `authentik-internal` (`10.0.7.0/24`), usada pelos outposts
para trocar o código OAuth por tokens. Isso preserva o HTTPS informado pelos
outposts e evita rejeitar o token com `InvalidIssuer`. Ajuste essas configurações
se as sub-redes mudarem. As portas HTTP/HTTPS são vinculadas
explicitamente a IPv4 para compatibilidade com as redes atuais do Swarm.

## Home Assistant

O Home Assistant usa autenticação nativa no navegador e no app Companion,
com CrowdSec no Traefik. O outpost dedicado foi retirado. A configuração está
no [README do serviço](../homeassistant/README.md).

## Cadastro do FichasRPG

O portal permite cadastro aberto por senha, com confirmação por e-mail, ou pelo
Google. As contas novas recebem somente `fichasrpg-users`. O provisionamento,
as políticas do Google e os testes ficam em
[`fichasrpg-signup/README.md`](fichasrpg-signup/README.md).

## Logout do Fichas RPG

O provedor `fichamutantes-forward-auth` usa `fichasrpg-invalidation-flow`:
o estágio User Logout encerra a sessão do Authentik e um Redirect estático
inicia uma nova autenticação pelo endpoint `/outpost.goauthentik.io/start`,
sem carregar a home do portal antes da tela de login do Fichas RPG.
O fluxo padrão da marca e os provedores dos outros serviços são preservados.
A configuração fica salva no banco do Authentik. Pelo painel administrativo,
em **Flows and Stages → Flows**, o fluxo contém User Logout na ordem 0 e
Redirect na ordem 100, com destino
`https://fichasrpg.nilvanlopes.com/outpost.goauthentik.io/start?rd=https%3A%2F%2Ffichasrpg.nilvanlopes.com%2F`.
Esse destino é reaplicado por `python3 branding/apply-theme.py --apply`.
Em **Applications → Providers**, esse fluxo é selecionado como fluxo de
invalidação apenas do provedor `fichamutantes-forward-auth`.

A brand **Fichas RPG** é separada da brand padrão do Authentik e define
`pt-BR` somente nas páginas dos fluxos do FichasRPG. A configuração e o CSS da
brand padrão são preservados. O provisionamento e a seleção da brand pelo
Traefik estão documentados em [`branding/README.md`](branding/README.md).

O botão Sair continua passando pelo outpost para também remover sua sessão.
A URL de retorno é fixa; parâmetros fornecidos pelo navegador não definem
o destino após o logout.

No fluxo `fichasrpg-authentication-flow`, o vínculo do estágio de identificação
usa a política `fichasrpg-return-authenticated-to-home`, avaliada quando o
estágio é executado. Se a sessão já estiver autenticada e não houver uma
reautenticação em andamento, a política usa o cancelamento nativo do fluxo e
a abertura da aplicação `fichamutantes`, cujo destino é o portal. Isso limpa
o fluxo pendente antes do retorno, preservando o logout. Assim, voltar ao login
leva ao portal sem reutilizar o callback OAuth anterior. A política fica salva
no Authentik, em **Customization → Policies**, e é vinculada somente ao fluxo
do Fichas RPG.

This repository contains a `docker-compose.yml` file for setting up [Authentik](https://goauthentik.io/), an open-source identity provider, along with PostgreSQL. This setup is configured to work with Traefik for reverse proxying and SSL termination.

## Services Included

*   **`postgresql`**: A PostgreSQL database instance used by Authentik.
*   **`server`**: The main Authentik application server.
*   **`worker`**: The Authentik worker process for background tasks.

## Prerequisites

Before you begin, ensure you have the following installed:

*   [Docker](https://docs.docker.com/get-docker/)
*   [Docker Compose](https://docs.docker.com/compose/install/) (usually comes with Docker Desktop)
*   A Traefik instance configured with a `traefik-public` network and a `cloudflare` certresolver, as indicated by the labels in the `server` service.

## Configuration

This setup relies on environment variables defined in a `.env` file at the root of this directory. An example `.env` file is provided, which you should copy and fill out.

**`.env` variables to configure:**

*   `PG_PASS`: Password for the PostgreSQL user.
*   `PG_USER`: (Optional) PostgreSQL username. Defaults to `authentik`.
*   `PG_DB`: (Optional) PostgreSQL database name. Defaults to `authentik`.
*   `AUTHENTIK_SECRET_KEY`: A strong, random secret key for Authentik.
*   `AUTHENTIK_BOOTSTRAP_PASSWORD`: The initial password for the `akadmin` user in Authentik.
*   `AUTHENTIK_LOG_LEVEL`: (Optional) Log level for Authentik. Defaults to `info`.
*   `AUTHENTIK_EMAIL__HOST`: SMTP host for email sending.
*   `AUTHENTIK_EMAIL__PORT`: SMTP port (default 587).
*   `AUTH_EMAIL_USER`: Username for SMTP authentication.
*   `AUTHENTIK_EMAIL__PASSWORD`: Password for SMTP authentication.
*   `AUTH_EMAIL_FROM`: Sender email address.
*   `AUTHENTIK_HOST`: The name to access authentik in browser

**Example `.env` (create this file):**

```dotenv
PG_PASS=your_postgres_password
AUTHENTIK_SECRET_KEY=a_very_long_and_random_string_of_characters
AUTHENTIK_BOOTSTRAP_PASSWORD=your_initial_admin_password
AUTHENTIK_EMAIL__HOST=smtp.example.com
AUTH_EMAIL_USER=your_smtp_username
AUTHENTIK_EMAIL__PASSWORD=your_smtp_password
AUTH_EMAIL_FROM=authentik@example.com
AUTHENTIK_HOST=authentik.yourdomain.com
```

## Usage

1.  **Create your `.env` file**: Copy the example above and fill in your desired values.
2.  **Start the services**:
    ```bash
    docker-compose up -d
    ```
    This command will pull the necessary Docker images, create the volumes and networks, and start all services in detached mode.

3.  **Check logs (optional)**:
    ```bash
    docker-compose logs -f
    ```
    This will show the logs from all services, which can be useful for debugging.

4.  **Stop the services**:
    ```bash
    docker-compose down
    ```
    This will stop and remove the containers, but keep the volumes.

5.  **Stop and remove everything (including data volumes)**:
    ```bash
    docker-compose down -v
    ```
    **Use this with caution as it will delete your database data!**

## Accessing Authentik

The `server` service is exposed via Traefik. Based on the `traefik.http.routers.authentik.rule` label in the `docker-compose.yml`, Authentik should be accessible at:

`https://authentik.yourdomain.com`

Make sure your DNS is configured correctly to point `authentik.yourdomain.com` to your Traefik instance.

## Networks and Volumes

**Networks:**

*   `traefik-public`: An external network used for Traefik integration.
*   `authentik-internal`: An internal overlay network for communication between Authentik services.

**Volumes:**

*   `authentik-database`: Stores PostgreSQL data.
*   `authentik-media`: Stores Authentik media files.
*   `authentik-certs`: Stores Authentik certificates.
*   `authentik-templates`: Stores Authentik templates.
