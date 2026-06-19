# Ch04 — Migração: Auth, CaC, APIs e RPM→Container

## Migração de LDAP/SAML/RADIUS para o Gateway

Todo config de 3rd party auth que estava no Controller precisa ser refeito no Gateway:

```
Antes (2.6 Controller): Settings → Authentication → LDAP Settings
Agora (2.7 Gateway): Access Management → Authentication Methods → Add Authentication
```

### Exemplo: Migrar LDAP do Controller para Gateway

```bash
# 1. Exportar config atual do Controller (antes do upgrade)
curl https://controller.empresa.org/api/controller/v2/settings/ldap/ \
  -H "Authorization: Bearer <token>" > ldap-config-backup.json

# 2. Após upgrade, reconfigurar no Gateway via UI ou API
curl -X POST https://aap.empresa.org/api/gateway/v1/authenticators/ \
  -H "Authorization: Bearer <admin_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "LDAP Corporativo",
    "type": "ansible_base.authentication.backend.ldap.LDAPBackend",
    "configuration": {
      "SERVER_URI": ["ldaps://ldap.empresa.org:636"],
      "BIND_DN": "cn=svc-aap,ou=services,dc=empresa,dc=org",
      "BIND_PASSWORD": "<senha>",
      "USER_SEARCH": [...],
      "GROUP_SEARCH": [...]
    }
  }'
```

## Migração de CaC para Gateway URLs

```yaml
# ANTES (2.6) — URL do Controller diretamente
- name: Criar Job Template
  ansible.controller.job_template:
    name: "Deploy App"
    ...
    controller_host: "https://controller.empresa.org"   # ← URL direta
    controller_oauthtoken: "{{ controller_token }}"     # ← token do Controller

# DEPOIS (2.7) — Via Gateway
- name: Criar Job Template
  ansible.platform.job_template:           # ← nova collection
    name: "Deploy App"
    ...
    controller_host: "https://aap.empresa.org"          # ← Gateway URL
    controller_oauthtoken: "{{ gateway_token }}"        # ← token do Gateway
```

### Variáveis de conexão atualizadas

| Variável antiga | Variável nova |
|---|---|
| `controller_host: controller.empresa.org` | `controller_host: aap.empresa.org` (Gateway) |
| `controller_oauthtoken:` (token controller) | `controller_oauthtoken:` (token Gateway) |
| `ah_host: hub.empresa.org` | `ah_host: aap.empresa.org` (Gateway) |
| `ah_token:` (token hub) | `ah_token:` (token Gateway) |

## Migração de Scripts e Pipelines CI/CD

```bash
# ANTES: acessar controller diretamente
curl https://controller.empresa.org/api/v2/job_templates/ \
  -H "Authorization: Bearer <controller_token>"

# DEPOIS: acessar via Gateway (URL diferente + novo token)
curl https://aap.empresa.org/api/controller/v2/job_templates/ \
  -H "Authorization: Bearer <gateway_token>"

# Reautenticar container registry
podman login aap.empresa.org \
  -u <gateway_username> \
  -p <gateway_token>
```

## Migração RPM → Container (2.6 → 2.7)

```
RPM 2.6 → Container 2.7 é suportado.
RPM 2.6 → Operator 2.7 é suportado.
RPM 2.7 → NÃO EXISTE (RPM foi removido).
```

Processo geral:
1. Instalar novo ambiente Container/Operator 2.7 (servidor novo)
2. No ambiente 2.6 RPM: `ansible-playbook -i inventory backup.yml`
3. Copiar backup para o novo servidor
4. Importar no novo ambiente: `ansible-playbook -i inventory restore.yml -e backup_file=...`
5. Validar dados (jobs, inventários, credenciais)
6. Redirecionar DNS/load balancer para novo servidor

## Mudanças na API — Referência Rápida

| Componente | 2.6 endpoint | 2.7 (somente via Gateway) |
|---|---|---|
| Controller | `/api/controller/v2/` | `https://gateway.host/api/controller/v2/` |
| Hub | `/api/galaxy/v1/` | `https://gateway.host/api/galaxy/v1/` |
| EDA | `/api/eda/v1/` | `https://gateway.host/api/eda/v1/` |
| Gateway | `/api/gateway/v1/` | `https://gateway.host/api/gateway/v1/` |

> Headers de autenticação: sempre `Authorization: Bearer <gateway_token>` no 2.7.
