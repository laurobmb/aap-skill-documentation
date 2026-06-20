# Ch03 — Autenticação na API

## Métodos de Autenticação Disponíveis

| Método | Quando usar |
|---|---|
| **OAuth2 Token (PAT)** | **Recomendado** — scripts, CI/CD, integrações programáticas |
| **Session** | Browser — UI ou API browser interativo |
| **Basic** | curl simples, testes rápidos (desabilitar em produção) |
| **SSO** | Login federado (LDAP, SAML, OIDC, GitHub, etc.) |

## OAuth2 — Criar Token (PAT)

```bash
# Criar token via API (autenticar com user:password uma única vez):
curl -u admin:senha -k -X POST \
  https://<gateway>/api/gateway/v1/tokens/ \
  -H "Content-Type: application/json" \
  -d '{"description": "token-ci-cd", "scope": "write"}'

# Resposta:
{
  "id": 42,
  "token": "eyJhbGciOi...",   ← guardar este valor
  "scope": "write",
  "expires": "2025-12-31T00:00:00Z"
}
```

**Escopos disponíveis:**
```
read   → apenas leitura
write  → leitura e escrita (recomendado para automação)
```

## OAuth2 — Usar Token em Requests

```bash
# Header Authorization: Bearer
curl -k -X GET \
  -H "Authorization: Bearer <token>" \
  https://<gateway>/api/controller/v2/jobs/

# Com dados JSON:
curl -k -X POST \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"limit": "webservers"}' \
  https://<gateway>/api/controller/v2/job_templates/14/launch/
```

## OAuth2 — Criar Token via UI

```
Access Management → OAuth Applications → Tokens → Create Token

ou (simpler):
Platform Gateway API URL/api/gateway/v1/users/<user_id>/tokens/
```

## Revogar Token

```bash
# Revogar token específico:
curl -k -X DELETE \
  -H "Authorization: Bearer <admin-token>" \
  https://<gateway>/api/gateway/v1/tokens/<token_id>/

# Revogar todos os tokens de um usuário (awx-manage):
awx-manage revoke_oauth2_tokens --user <username>
```

## Basic Authentication

```bash
# flag --user adiciona header Authorization: Basic automaticamente
curl -X GET --user 'admin:senha' \
  https://<gateway>/api/gateway/v1/tokens/ -k -L
```

> ⚠️ **Não recomendado em produção** — enviar credenciais em cada request. Preferir OAuth2.

## Desabilitar Basic Auth

```
Settings → Platform gateway → Edit platform gateway settings
  → ☐ Gateway basic auth enabled → Save
```

## Habilitar Tokens para Usuários SSO (externos)

```
# Por padrão, usuários SSO/externos não podem gerar tokens OAuth2.
# Para habilitar:
Settings → Platform gateway → Edit platform gateway settings
  → ☑ Allow external users to create OAuth2 tokens → Save
```

## Session Authentication (Browser)

```bash
# 1. GET para obter o csrftoken:
curl -k -c - https://<gateway>/api/gateway/v1/login/
# Copiar o valor do csrftoken retornado

# 2. POST para login:
curl -X POST \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  --referer https://<gateway>/api/gateway/v1/login/ \
  -H 'X-CSRFToken: <csrftoken>' \
  --data 'username=admin&password=senha' \
  --cookie 'csrftoken=<csrftoken>' \
  https://<gateway>/api/gateway/v1/login/ -k -D - -o /dev/null

# 3. Usar com cookie:
curl -X GET -H 'Cookie: awx_sessionid=<session-id>;' \
  https://<gateway>/api/controller/v2/jobs/ -k
```

> Session timeout padrão: 1800s. Ajustar via `SESSION_COOKIE_AGE`.
