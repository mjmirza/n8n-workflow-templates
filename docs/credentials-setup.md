# Credentials setup

n8n stores secrets in credentials, separate from the workflow. A workflow node records the credential's name and id. The secret itself stays in the encrypted credential store. Templates in this repository follow that split: they tell you which credential type to create, and they do not contain tokens, passwords, or private keys.

Create credentials in the same n8n project where the workflow will run. A workflow in one project cannot use a credential that is private to another project.

## Create a credential from a node

1. Open the node that needs to call the external service.
2. Find **Credential to connect with**.
3. Choose **Create new credential**.
4. Name it so the environment is obvious later. `Slack (production)` and `Slack (sandbox)` are safer than `Slack account` once you have more than one.
5. Fill in the fields the credential form asks for. For an API token, paste the token only into that form.
6. Save the credential, then save the workflow.

Use the credential type the node offers. For a service n8n already knows, prefer that predefined type over generic Header Auth. The predefined type already knows which header or OAuth flow the service expects, and other nodes for the same service can reuse it.

## Generic credentials for a simple HTTP API

When the service has no dedicated node, use the **HTTP Request** node and a generic credential:

| Service asks for | Credential type | What you enter |
| --- | --- | --- |
| `Authorization: Bearer <token>` | Header Auth | Name `Authorization`, value `Bearer` followed by the token |
| A key in the query string | Query Auth | The parameter name and the key |
| Username and password | Basic Auth | The username and password |
| An OAuth2 access token you refresh yourself | OAuth2 API | Client id, client secret, and the token and auth URLs from the provider |

The Header Auth value includes the word `Bearer` and a single space when the vendor documents that scheme. Do not also add an `Authorization` header on the node, or you will send two and the request will fail in a way that looks like a bad token.

## OAuth redirects on a self-hosted instance

OAuth credentials need a redirect URL that the provider can reach. n8n shows the URL in the credential dialog. It is built from the public base URL of the instance.

On a self-hosted instance, set that base URL before you connect OAuth apps. n8n 2.30.0 and later read `N8N_WEBHOOK_URL`. Earlier versions read `WEBHOOK_URL`. Use the public `https` origin with a trailing slash, for example `https://n8n.example.com/`. If this value is `http://localhost:5678/` while you are connecting a provider that only allows HTTPS callbacks, the provider will reject the redirect.

Copy the redirect URL from the credential dialog into the provider's app settings exactly. A trailing-slash mismatch is enough to make the callback fail.

After you save the provider settings, use the credential's **Connect** or **Sign in** button and finish the browser prompt. Save the credential again when n8n reports that the account is connected.

## Check that the credential works

Execute the node once with data that the service will accept as a dry run: read a record, or post into a sandbox channel. A successful node output means the credential authenticated. An `401` or `403` means the token, the scopes, or the account permissions are wrong. An `404` usually means the URL or the resource id is wrong, not the credential.

If the node has no safe read operation, call a documented health or "who am I" endpoint with the HTTP Request node and the same credential before you wire the real action.

## Share access without copying secrets

When several workflows need the same account, create the credential once and select it on each node. In a team project, share the credential with the project instead of pasting the token into a second credential. Revoking access is then a single change.

To rotate a token:

1. Issue a new token at the provider.
2. Edit the existing credential and replace the secret, or create a second credential if you need a rollback.
3. Run one manual execution that uses the new secret.
4. Revoke the old token at the provider after that run succeeds.

## What belongs in a template README

Document the credential by type and by purpose. Write "Slack API credential with `chat:write` on the target channel", not the token. Name every node that uses it. If two nodes must share one account, say so, because picking two different credentials is a common import mistake.

Do not export credentials into git. `n8n export:credentials` writes encrypted credential records for that instance. Those files are not templates, and they do not belong in this repository.
