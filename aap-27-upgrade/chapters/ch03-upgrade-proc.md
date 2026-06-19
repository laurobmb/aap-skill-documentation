# Ch03 — Procedimento de Upgrade

## Upgrade Containerizado (2.6 → 2.7)

```bash
# 1. Baixar novo installer 2.7
# Acesse: https://access.redhat.com/downloads/content/480

# 2. Extrair
tar -xzf ansible-automation-platform-setup-2.7.tar.gz
cd ansible-automation-platform-setup-2.7

# 3. Copiar inventory file existente (do 2.6) para o novo diretório
cp /opt/ansible-automation-platform-2.6/installer/inventory .

# 4. Executar upgrade (idempotente, usa o mesmo install.yml)
ansible-playbook -i inventory install.yml

# 5. Verificar versão após upgrade
curl -s https://aap.exemplo.org/api/gateway/v1/ping/ \
  -H "Authorization: Bearer <token>" | jq
```

## Upgrade OCP/Operator (2.6 → 2.7)

### Via OperatorHub (Canal)

```bash
# Verificar canal atual do Subscription
oc get subscription ansible-automation-platform-operator -n aap-operator \
  -o jsonpath='{.spec.channel}'
# → stable-2.6

# Alterar canal para 2.7
oc patch subscription ansible-automation-platform-operator \
  -n aap-operator \
  --type merge \
  -p '{"spec": {"channel": "stable-2.7"}}'

# Acompanhar o upgrade
oc get clusterserviceversion -n aap-operator -w
```

### In-Channel Upgrade (patch minor)

```bash
# Upgrade automático (InstallPlan aprovado automaticamente se approval=Automatic)
oc get installplan -n aap-operator

# Ou aprovar manualmente
oc patch installplan <installplan-name> -n aap-operator \
  --type merge -p '{"spec": {"approved": true}}'
```

## Upgrade de Componentes Extras

### Developer Hub Plugins (Helm)

```bash
# 1. Atualizar registry de plugins
helm repo update

# 2. Ver versão atual
helm list -n rhdh

# 3. Upgrade do Helm release
helm upgrade <release-name> redhat-developer-hub/backstage \
  --namespace rhdh \
  -f values.yaml \
  --version <nova_versao>
```

### Automation Dashboard

```bash
# Verificar versão compatível
# dashboard 0.4.x → compatível com AAP 2.7

# Upgrade (re-executar installer com nova versão)
tar -xzf ansible-automation-dashboard-setup-0.4.tar.gz
cd ansible-automation-dashboard-setup-0.4
ansible-playbook -i inventory install.yml
```

### Ansible Automation Portal

```bash
# Verificar compatibilidade: portal 2.2.x → AAP 2.7

# Upgrade via Helm OCI (novo método a partir do portal 2.x)
helm upgrade <release-name> \
  oci://registry.redhat.io/ansible-automation-portal/helm-chart:2.2 \
  --namespace portal \
  -f values.yaml
```

## Rollback (Pós-Upgrade)

```bash
# Se necessário reverter (containerizado)
# Restore do backup feito ANTES do upgrade:
cd /opt/ansible-automation-platform-setup-2.6
ansible-playbook -i inventory restore.yml \
  -e backup_file=/var/lib/awx/backup-pre-upgrade.tar.gz
```

> Fazer backup ANTES do upgrade: `ansible-playbook -i inventory backup.yml`
