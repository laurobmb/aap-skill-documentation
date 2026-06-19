# Ch05 — Pós-Instalação e Subscription

## Registrar Subscription

```bash
# Após instalação, acessar o Gateway e fazer login como admin
# Navegar para: Administration → Subscription

# Opção 1: Credenciais Red Hat Network
# Usuário/senha Red Hat Customer Portal

# Opção 2: Manifest file
# Baixar manifest em: https://access.redhat.com/management/subscription_allocations
# Upload do manifest.zip na UI do AAP
```

## Registry Service Account (Offline/Air-gapped)

```ini
# No inventory file, usar service account do registry.redhat.io
# Criar em: https://access.redhat.com/RegistryAuthentication
registry_username=<service_account_name>
registry_password=<service_account_token>
```

## Seed de Coleções no Hub

```ini
# Habilitar seed automático de coleções certificadas Red Hat
hub_seed_collections=true
# ATENÇÃO: requer 32 GB RAM e pode demorar 45+ minutos
```

## Verificar Instalação

```bash
# Verificar containers rodando (containerizado)
podman ps --format "table {{.Names}}\t{{.Status}}"

# Verificar acesso ao Gateway
curl -k https://aap.exemplo.org/api/gateway/v1/ping/

# Verificar Controller
curl -k -u admin:<senha> https://aap.exemplo.org/api/controller/v2/ping/

# Verificar Hub
curl -k -u admin:<senha> https://aap.exemplo.org/api/galaxy/pulp/api/v3/status/

# Verificar EDA
curl -k -u admin:<senha> https://aap.exemplo.org/api/eda/v1/
```

## Comandos de Manutenção (Containerizado)

```bash
# Re-executar instalação (idempotente)
cd /opt/ansible-automation-platform/installer
ansible-playbook -i inventory install.yml

# Verificar logs de um serviço específico
podman logs aap-controller-web

# Reiniciar serviço
podman restart aap-controller-web

# Backup dos dados (containerizado)
ansible-playbook -i inventory backup.yml

# Restore
ansible-playbook -i inventory restore.yml -e backup_file=/path/to/backup.tar.gz
```
