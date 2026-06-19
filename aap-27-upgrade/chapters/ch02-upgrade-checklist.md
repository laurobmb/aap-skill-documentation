# Ch02 — Checklist Pré e Pós-Upgrade

## PRÉ-UPGRADE: O que verificar ANTES de atualizar para 2.7

### 1. Verificar versão atual

```bash
# Deve ser 2.6.x (2.4/2.5 precisam ir para 2.6 primeiro)
curl -s https://aap.exemplo.org/api/gateway/v1/ping/ \
  -H "Authorization: Bearer <token>" | jq .version
```

### 2. Detectar acesso direto a componentes (CRÍTICO)

Usar ferramenta oficial para escanear logs e identificar tráfego que bypass o Gateway:

```bash
# Coletar sos/must-gather primeiro

# Para instalação containerizada/RPM (via sos report)
uvx --from "git+https://github.com/ansible/aap-detect-direct-component-access" \
  aap-detect-direct-component-access /path/to/sosreport/

# Para OCP (via must-gather)
uvx --from "git+https://github.com/ansible/aap-detect-direct-component-access" \
  aap-detect-direct-component-access /path/to/must-gather/
```

> Requer `uvx` (`pip install uv`). Requer AAP 2.6 patch de 25/03/2026 ou posterior para os campos de log necessários.

### 3. Inventário pré-upgrade

Identificar e documentar:

```
☐ Scripts usando URLs diretas de componentes (controller.*, hub.*, eda.*)
☐ Pipelines CI/CD com tokens de componente
☐ Implementações CaC (ansible.controller, ansible.hub, ansible.eda)
☐ Personal Access Tokens ativos no Controller
☐ Integrações OAuth2/API com URLs de componentes
☐ Workflows: podman login / docker login no Hub
☐ Configurações de auth de 3rd party (LDAP, SAML, RADIUS, TACACS+) no Controller
☐ Usuários que só têm conta no Controller (sem conta no Gateway)
```

### 4. Criar contas no Gateway para todos os usuários

**CRÍTICO**: usuários sem conta no Gateway não conseguirão logar após o upgrade.

```bash
# Via API do Gateway: criar/verificar usuários
curl https://aap.exemplo.org/api/gateway/v1/users/ \
  -H "Authorization: Bearer <admin_token>"

# Instruir usuários a definir senha no Gateway antes do upgrade
# Gateway → Users → <usuário> → Set Password
```

### 5. Verificar/exportar configurações de auth

Documentar todas as configurações de LDAP/SAML do Controller para reconfigurar no Gateway:

```bash
# Exportar configuração LDAP do Controller
curl https://aap.exemplo.org/api/controller/v2/settings/ldap/ \
  -H "Authorization: Bearer <admin_token>"
```

## PÓS-UPGRADE: O que fazer DEPOIS de atualizar para 2.7

```
☑ Recriar todos os PATs via Gateway (tokens antigos de componente não funcionam)
☑ Atualizar URLs em scripts: *.componente.empresa.org → gateway.empresa.org
☑ Atualizar CaC: controller_host= → gateway URL
☑ Re-autenticar container registry: podman login gateway.empresa.org
☑ Reconfigurar LDAP/SAML/RADIUS no Gateway (não mais no Controller)
☑ Verificar autenticação: Access Management → Authentication Methods
☑ Testar login de usuários via cada autenticador
☑ Validar workflows CI/CD com novos tokens
```

## Validar Gateway após Upgrade

```bash
# Verificar que auth via Gateway funciona
curl -X POST https://aap.exemplo.org/api/gateway/v1/tokens/ \
  -u admin:<senha>

# Confirmar que acesso direto ao controller retorna 401 (esperado)
curl https://controller.empresa.org/api/controller/v2/ping/
# → 401 Unauthorized (CORRETO no 2.7)

# Confirmar que acesso via Gateway funciona
curl https://aap.empresa.org/api/controller/v2/ping/ \
  -H "Authorization: Bearer <novo_token>"
# → 200 OK
```
