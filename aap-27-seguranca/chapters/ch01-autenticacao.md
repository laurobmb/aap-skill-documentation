# Ch01 — Autenticação Centralizada no AAP 2.7

## Mudanças Críticas em 2.7

**Autenticação centralizada no Platform Gateway** — autenticação direta foi removida:

| Antes (2.6) | Agora (2.7) |
|---|---|
| Basic auth direta no Controller API | ❌ HTTP 401 — removido |
| Session auth direta no Controller | ❌ HTTP 401 — removido |
| Token a nível de componente (controller) | ❌ Removido |
| LDAP configurado por componente | ❌ Configurar apenas no Gateway |
| SAML/OAuth configurado por componente | ❌ Configurar apenas no Gateway |

**O que continua funcionando** (operações internas, sem passar pelo Gateway):
- Service-to-service (Controller buscando EE no Hub)
- EDA workers via WebSocket para Controller
- Container registry Hub: `podman login <platform-host> -u <gateway-user>`

## Fluxo de Autenticação 2.7

```
Usuário / API externa
        ↓
  Platform Gateway (único ponto de autenticação)
        ↓  emite JWT token
  Controller / Hub / EDA (aceitam apenas JWT do Gateway)
```

## Tipos de Autenticação Suportados (Pluggable)

| Tipo | Protocolo | Caso de uso |
|---|---|---|
| **Local** | Username/password interno | Contas locais, admin inicial |
| **LDAP** | LDAP/LDAPS | Active Directory, OpenLDAP |
| **SAML** | SAML 2.0 | SSO corporativo (Okta, ADFS, KeyCloak) |
| **Generic OIDC** | OIDC | Qualquer IdP OIDC |
| **Microsoft Entra ID** | OIDC/OAuth2 | Azure AD |
| **Google OAuth2** | OAuth2 | Google Workspace |
| **GitHub** | OAuth2 | GitHub.com |
| **GitHub Organization** | OAuth2 | Org específica do GitHub |
| **GitHub Team** | OAuth2 | Time específico do GitHub |
| **GitHub Enterprise** | OAuth2 | GitHub Enterprise Server |
| **GitHub Enterprise Org** | OAuth2 | Org no GitHub Enterprise |
| **GitHub Enterprise Team** | OAuth2 | Time no GitHub Enterprise |
| **Keycloak** | OIDC | Keycloak auto-hospedado |
| **RADIUS** | RADIUS | Autenticação de rede |
| **TACACS+** | TACACS+ | Dispositivos de rede Cisco |

## Configurar LDAP no Platform Gateway

```
Gateway → Authentication → Add Authentication → LDAP
```

Campos obrigatórios:
```yaml
Server URI: ldaps://ldap.empresa.org:636
Bind DN: cn=svc-aap,ou=service,dc=empresa,dc=org
Bind Password: <senha>
User Search:
  Base DN: ou=users,dc=empresa,dc=org
  Scope: SUBTREE
  Filter: (sAMAccountName=%(user)s)
Group Search:
  Base DN: ou=groups,dc=empresa,dc=org
  Scope: SUBTREE
  Filter: (objectClass=group)
```

## Configurar SAML no Platform Gateway

```
Gateway → Authentication → Add Authentication → SAML
```

```yaml
Entity ID: https://aap.empresa.org/api/gateway/v1/auth/saml/
SP Public Cert: <certificado público do AAP>
SP Private Key: <chave privada do AAP>
IdP Entity ID: <entity_id do IdP>
IdP Metadata URL: https://sso.empresa.org/realms/master/protocol/saml/descriptor
```

> Após configurar qualquer OAuth/SSO: atualizar Callback URL no IdP para:
> `https://aap.empresa.org/api/gateway/v1/auth/<tipo>/callback/`

## Autenticador Local — Habilitar/Desabilitar

```
Gateway → Authentication → Local → Toggle Enable/Disable
```

> **Cuidado:** desabilitar o autenticador local bloqueia todos os usuários locais (incluindo admin). Manter habilitado se não há outro método configurado.

## Configurar Timeout de Sessão

```yaml
# Em Gateway → Authentication → <Autenticador> → Advanced
Session Timeout: 3600        # segundos (padrão: 1800)
Token Expiry: 31536000       # segundos para tokens de longa duração
```
