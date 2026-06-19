# Ch05 — Tokens, OAuth2 e Administração

## Personal Access Tokens (PATs) no 2.7

> **Breaking change:** tokens a nível de componente (controller, hub) foram **removidos** no 2.7. Todos os tokens são criados via **Platform Gateway**.

```
Gateway → Access Management → Access Tokens → Create Token

# Token de componente (legado) → ❌ removido
# Token do Gateway → ✅ único método suportado no 2.7
```

```bash
# Criar PAT via API do Gateway
curl -X POST https://aap.exemplo.org/api/gateway/v1/tokens/ \
  -H "Authorization: Bearer <admin_token>" \
  -H "Content-Type: application/json" \
  -d '{"description": "CI/CD Jenkins", "application": null, "scope": "write"}'
```

**Expiração padrão:** 1 ano (mudança do 2.6 onde era 1000 anos)

```python
# Customizar expiração (settings.py ou inventory extra_settings)
OAUTH2_PROVIDER__ACCESS_TOKEN_EXPIRE_SECONDS = 31536000  # 1 ano
```

## OAuth2 Applications

```
Gateway → Access Management → OAuth Applications → Create OAuth Application

Grant types:
  - Authorization Code: para web apps externas (Jenkins, ServiceNow)
  - Password: para clientes com acesso nativo
```

```bash
# Usar token OAuth2 na API
curl https://aap.exemplo.org/api/controller/v2/job_templates/ \
  -H "Authorization: Bearer <oauth_token>"
```

## Gerenciamento via awx-manage / aap-gateway-manage

```bash
# Comandos do Controller (containerizado)
podman exec -it aap-controller-task awx-manage --help

# Criar superuser
podman exec -it aap-controller-task awx-manage createsuperuser

# Listar tokens
podman exec -it aap-controller-task awx-manage list_instances

# Revogar tokens OAuth2
podman exec -it aap-controller-task awx-manage revoke_oauth2_tokens

# Limpar sessões expiradas
podman exec -it aap-controller-task awx-manage clearsessions

# Limpar tokens expirados
podman exec -it aap-controller-task awx-manage cleartokens
```

```bash
# Comandos do Gateway
podman exec -it aap-gateway aap-gateway-manage --help

# Auditoria de emails modificados
podman exec -it aap-gateway aap-gateway-manage detect_changed_emails --audit
```

## Backup e Restore (Containerizado)

```bash
# Backup completo
cd /opt/ansible-automation-platform/installer
ansible-playbook -i inventory backup.yml
# Gera: /var/lib/awx/backup-<timestamp>.tar.gz

# Restore
ansible-playbook -i inventory restore.yml \
  -e backup_file=/var/lib/awx/backup-2025-01-15.tar.gz
```

## Backup no OpenShift (Operator)

```yaml
# Criar backup via CRD
apiVersion: aap.ansible.com/v1alpha1
kind: AnsibleAutomationPlatformBackup
metadata:
  name: meu-backup
  namespace: aap
spec:
  deployment_name: myaap
  backup_pvc: aap-backup-pvc        # PVC para armazenar o backup
```

```yaml
# Restore via CRD
apiVersion: aap.ansible.com/v1alpha1
kind: AnsibleAutomationPlatformRestore
metadata:
  name: meu-restore
  namespace: aap
spec:
  deployment_name: myaap
  backup_name: meu-backup
```

## Timeout de Rotas do Gateway

Ajustar para uploads grandes (collections, imagens de container):

```yaml
# Para deployment containerizado (inventory extra_settings)
gateway_extra_settings:
  - setting: RECEPTOR_TCP_ESTABLISHED_TIMEOUT
    value: 7200

# Para operações Lightspeed
# gateway_lightspeed_route_timeout: 300  (padrão)
```

| Operação | Timeout recomendado |
|---|---|
| Upload de collection pequena (< 100 MB) | 60s |
| Upload de collection grande | 300s |
| Push de imagem container | 600s |
| Lightspeed inference | 120–300s |
