# Ch03 — Hardening e Compliance

## Princípios de Hardening AAP

1. **Princípio do mínimo privilégio** — usuários e service accounts com apenas as permissões necessárias
2. **Criptografia em trânsito** — TLS em todas as comunicações, sem HTTP puro em produção
3. **Rotação de credenciais** — integrar com CyberArk, HashiCorp Vault ou ESO
4. **Audit logging** — manter Activity Stream acessível e retido
5. **Inventory seguro** — nunca texto claro em Git (Ansible Vault obrigatório)

## Hardening do Inventory File

```ini
# Desabilitar HTTP (forçar HTTPS)
[all:vars]
controller_nginx_disable_https=false     # manter HTTPS ligado
hub_nginx_disable_https=false
eda_nginx_disable_https=false

# TLS customizado (evitar auto-assinados)
ca_tls_cert=/etc/pki/aap/ca.crt
ca_tls_key=/etc/pki/aap/ca.key

# mTLS para PostgreSQL externo
controller_pg_sslmode=verify-full
controller_pg_sslrootcert=/etc/pki/aap/pg-ca.crt
```

## Configurações de Segurança na UI

```
Platform Gateway → Settings → Security:
├── ALLOW_OAUTH2_FOR_EXTERNAL_USERS: false   # bloquear tokens OAuth2 para LDAP users
├── CSRF_TRUSTED_ORIGINS: ['https://gateway.empresa.org']
└── SECURE_HSTS_SECONDS: 31536000           # HSTS 1 ano
```

## Gestão de Segredos com HashiCorp Vault

```yaml
# Credential Type: HashiCorp Vault Secret Lookup
# Configurar Vault Credential no Controller:
vault_addr: "https://vault.empresa.org"
vault_token: !vault |
      $ANSIBLE_VAULT;1.1;AES256...

# Usar em Job Templates via lookup
- name: Obter senha do banco
  set_fact:
    db_password: "{{ lookup('hashi_vault', 'secret/path/db password=token vault_addr=https://vault.empresa.org') }}"
```

## External Secrets Operator (ESO) — OpenShift

Para instalações Operator no OpenShift, integrar ESO para rotação automática de segredos:

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: aap-controller-credentials
  namespace: aap-production
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: ClusterSecretStore
  target:
    name: aap-controller-credentials
    creationPolicy: Owner
  data:
    - secretKey: controller_password
      remoteRef:
        key: aap/controller
        property: admin_password
```

## Isolamento de Execução com Instance Groups

Segmentar execution nodes por nível de sensibilidade:

```
Instance Group "prod-critical"    → apenas Execution Nodes dedicados à produção
Instance Group "dev-sandbox"      → Execution Nodes isolados para desenvolvimento
Instance Group "dmz-network"      → Execution Nodes na DMZ para ativos de rede
```

```yaml
# Job Template: restringir a Instance Group específico
- ansible.platform.job_template:
    name: "Backup Rede"
    instance_groups:
      - "dmz-network"
```

## Compliance — Checklist CIS AAP

```
[ ] TLS habilitado em todos os endpoints (sem HTTP puro)
[ ] Certificados válidos (não auto-assinados) em produção
[ ] Ansible Vault para todos os segredos no inventory
[ ] Rotação automática de credenciais via Vault/CyberArk
[ ] Autenticação LDAP/SAML integrada (não usar apenas local)
[ ] Break-glass account documentado e guardado em cofre
[ ] Activity Stream habilitado e retido por >= 90 dias
[ ] Execution Nodes isolados por sensibilidade (Instance Groups)
[ ] Acesso SSH aos nós AAP restrito a IPs de gestão
[ ] Podman containers rodando como rootless (padrão)
[ ] PostgreSQL com TLS e `sslmode=verify-full`
[ ] Redis com TLS habilitado
[ ] Receptor (Mesh) com certificados assinados pela CA interna
```

## Receptor — Segurança do Automation Mesh

O Receptor usa TLS para todas as comunicações. As configurações de certificado são gerenciadas automaticamente pelo instalador, mas podem ser customizadas:

```ini
# Inventory — receptor com CA customizada
receptor_tls_cert=/path/to/receptor.crt
receptor_tls_key=/path/to/receptor.key
receptor_ca_cert=/path/to/receptor-ca.crt

# Signing keys para verificar integridade de trabalhos
receptor_signing_private_key=/path/to/signing.key
receptor_signing_public_key=/path/to/signing.pub
```
