# Ch01 — Tokens de Autenticação e Remotos

## Autenticação no Hub (AAP 2.7)

> **Mudança no 2.7:** Não é mais possível acessar a API do Hub diretamente. Toda autenticação passa pelo **Platform Gateway**.

```
Automation hub não suporta:
  ✗ Basic authentication (username/password direto)
  ✗ Service accounts
  ✓ Token management via Gateway
```

## Offline Token (para console.redhat.com / Keycloak)

Usado para sincronizar conteúdo de `console.redhat.com` para o Hub local:

```
# Obter offline token:
1. Acesse console.redhat.com
2. Automation Hub → Connect to Hub
3. Offline token → Load Token → Copy to clipboard

# Guardar de forma segura — expira após 30 dias de inatividade!
# Renovar antes de expirar:
curl https://sso.redhat.com/auth/realms/redhat-external/protocol/openid-connect/token \
  -d grant_type=refresh_token \
  -d client_id="cloud-services" \
  -d refresh_token="{{ offline_token }}" \
  --fail --silent --show-error --output /dev/null
```

## Configurar Remote rh-certified (Red Hat Certified Collections)

```
Automation Content → Remotes → rh-certified → Edit remote

Campos:
  URL:   Sync URL de console.redhat.com
         (ex: https://console.redhat.com/api/automation-hub/content/published/)
  Token: Token offline de console.redhat.com

→ Save remote
```

**Limitar quais coleções sincronizar (requirements.yml):**

```yaml
# Em vez de sincronizar tudo, usar requirements file no remote
collections:
  - name: ansible.netcommon
    version: ">=5.0.0"
  - name: community.vmware
  - name: redhat.satellite
    version: ">=3.2.0"
  - name: ansible.posix
```

> Sincronizar tudo pode consumir muito disco. Use requirements.yml para controle granular.

## Configurar Remote community (Ansible Galaxy)

```
Automation Content → Remotes → Community → Edit remote

YAML requirements (colar conteúdo do requirements.yml):
  collections:
    - name: community.aws
      version: 5.2.0
      source: https://galaxy.ansible.com
    - name: community.windows

→ Save remote
```

## Configurar Proxy para Remotos (Hub atrás de proxy)

```
Automation Content → Remotes → [remoto desejado] → Edit remote
→ Show advanced options

  Proxy URL:      http://proxy.empresa.org:3128
  Proxy Username: (se autenticado)
  Proxy Password: (se autenticado)
```

## Via ansible-galaxy CLI — Configurar Hub como servidor

```ini
# ~/.ansible/galaxy_token (ou galaxy_server_list no ansible.cfg)
[galaxy]
server_list = my_org_hub

[galaxy_server.my_org_hub]
url=https://aap.empresa.org/api/galaxy/
token=<gateway_token>
```

```bash
# Instalar coleção do Hub privado
ansible-galaxy collection install redhat.satellite --server my_org_hub

# Com requirements.yml
ansible-galaxy collection install -r requirements.yml --server my_org_hub
```
