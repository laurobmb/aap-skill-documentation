# Cheatsheet — AAP 2.7 Segurança

## Breaking Changes 2.7

```
Auth direta a controller/hub/EDA  → REMOVIDA (HTTP 401)
Component-level tokens            → REMOVIDOS
PATs criados via Gateway          → ÚNICO método
Token expiração padrão            → 1 ano (antes: 1000 anos)
LDAP/SAML/OIDC                    → Configurar APENAS no Gateway
```

## Tipos de Auth Suportados

```
Local | LDAP | SAML | Generic OIDC | Microsoft Entra ID
Google OAuth2 | Keycloak | GitHub | GitHub Org | GitHub Team
GitHub Enterprise | GitHub Enterprise Org | GitHub Enterprise Team
RADIUS | TACACS+
```

## Authenticator Map — Referência Rápida

| Map Type | Controla |
|---|---|
| Allow | Acesso ao sistema |
| Organization | Org do usuário |
| Team | Time do usuário |
| Role | Role atribuída |
| Is Superuser | Superuser sim/não |

| Trigger | Quando dispara |
|---|---|
| Always | Sempre |
| Never | Nunca |
| Group | Grupo do IdP |
| Attribute | Atributo do usuário |

## Criar PAT via API

```bash
curl -X POST https://aap.exemplo.org/api/gateway/v1/tokens/ \
  -H "Authorization: Bearer <token>" \
  -d '{"description": "CI", "scope": "write"}'
```

## awx-manage Essencial

```bash
# Prefixo: podman exec -it aap-controller-task awx-manage
createsuperuser
revoke_oauth2_tokens
cleartokens
clearsessions
list_instances
```

## RBAC Roles Built-in

```
Admin / Member / Execute / Use / Read / Auditor
```

## Compartilhar Credencial Entre Orgs

```
Credencial na Org A → Assign role "Use" → Org B / Team B
(Org B usa mas não vê a senha)
```
