# Ch02 — Inventory File e Variáveis de Instalação Containerizada

## Inventory File — Container Growth (1 VM)

```ini
# Container Growth Topology — 1 VM com todos os componentes

[automationgateway]
aap.example.org

[automationcontroller]
aap.example.org

[automationhub]
aap.example.org

[automationeda]
aap.example.org

[database]
aap.example.org

[all:vars]
# Conexão (self-contained local = connection local)
ansible_connection=local

# Banco de dados
postgresql_admin_username=postgres
postgresql_admin_password=<defina>

# Registry Red Hat
registry_username=<RHN username>
registry_password=<RHN password>

# Redis (standalone para growth, sentinel para HA)
redis_mode=standalone

# Platform Gateway
gateway_admin_password=<defina>
gateway_pg_host=aap.example.org
gateway_pg_password=<defina>

# Automation Controller
controller_admin_password=<defina>
controller_pg_host=aap.example.org
controller_pg_password=<defina>
controller_percent_memory_capacity=0.5

# Automation Hub
hub_admin_password=<defina>
hub_pg_host=aap.example.org
hub_pg_password=<defina>
hub_seed_collections=false   # true demora 45+ min e requer 32 GB RAM

# Event-Driven Ansible
eda_admin_password=<defina>
eda_pg_host=aap.example.org
eda_pg_password=<defina>
```

> SSH keys só são necessárias para instalação em hosts remotos. Para VM local use `ansible_connection=local`.

---

## Inventory File — Container Enterprise (11 VMs, exemplo simplificado)

```ini
[automationgateway]
gw1.exemplo.org
gw2.exemplo.org

[automationcontroller]
ctrl1.exemplo.org
ctrl2.exemplo.org

[automationhub]
hub1.exemplo.org
hub2.exemplo.org

[automationeda]
eda1.exemplo.org
eda2.exemplo.org

[execution_nodes]
hop1.exemplo.org    node_type=hop
exec1.exemplo.org   node_type=execution
exec2.exemplo.org   node_type=execution

[all:vars]
ansible_user=ansible
ansible_become=true

registry_username=<RHN username>
registry_password=<RHN password>

postgresql_admin_username=postgres
postgresql_admin_password=<defina>

# PostgreSQL externo
controller_pg_host=db.exemplo.org
controller_pg_password=<defina>
hub_pg_host=db.exemplo.org
hub_pg_password=<defina>
gateway_pg_host=db.exemplo.org
gateway_pg_password=<defina>
eda_pg_host=db.exemplo.org
eda_pg_password=<defina>

gateway_admin_password=<defina>
controller_admin_password=<defina>
hub_admin_password=<defina>
eda_admin_password=<defina>

redis_mode=standalone
# Para Redis HA: redis_mode=sentinel
```

---

## Redis: Standalone vs HA

| Variável | Valor | Quando usar |
|---|---|---|
| `redis_mode` | `standalone` | Growth topology, desenvolvimento, sem HA |
| `redis_mode` | `sentinel` | Enterprise topology com Redis HA (6 VMs adicionais) |

- Redis **pode ser colocado** junto com gateway, controller, hub ou eda
- Redis **não pode** ficar em execution nodes ou no host do PostgreSQL
- **Redis externo não é suportado** no containerizado (diferente do Operator)

---

## PostgreSQL Externo — Requisitos

```ini
# Variáveis para PostgreSQL externo
<componente>_pg_host=db.empresa.org
<componente>_pg_port=5432          # padrão
<componente>_pg_database=<nome>    # padrão: awx, hub, etc.
<componente>_pg_username=<user>    # padrão: awx, hub, etc.
<componente>_pg_password=<senha>
<componente>_pg_sslmode=prefer     # prefer | require | verify-full
```

Requisitos do PostgreSQL externo:
- Versão: 15, 16 ou 17
- Extensão ICU obrigatória
- PostgreSQL 16 ou 17: backup/restore deve ser feito com ferramentas nativas (não pelo AAP)

---

## Variáveis Principais por Componente

| Prefixo | Componente |
|---|---|
| `gateway_*` | Platform Gateway (nginx, rotas, SSL) |
| `controller_*` | Automation Controller |
| `hub_*` | Private Automation Hub |
| `eda_*` | Event-Driven Ansible |
| `postgresql_*` | Banco de dados gerenciado |
| `redis_*` | Redis |
| `registry_*` | Red Hat container registry |

Consultar `Inventory file variables` na documentação para lista completa (referência em aap-27-upgrade/ch04-referencia).
