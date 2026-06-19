# Cheatsheet — AAP 2.7 Instalação

## Breaking Changes 2.7
```
RPM installer → REMOVIDO (deprecated desde 2.5)
Acesso direto (sem gateway) → BLOQUEADO
```

## Topologias Rápidas

| Cenário | Topologia | VMs |
|---|---|---|
| POC / início | Container Growth | 1 VM (16GB/4CPU/60GB) |
| Produção HA | Container Enterprise | 11+ VMs |
| OpenShift pequeno | Operator Growth (SNO) | 32GB/16CPU/128GB |
| OpenShift produção | Operator Enterprise | Cluster 3+2 nodes |

## Inventory File — Variáveis Obrigatórias

```ini
[all:vars]
ansible_connection=local              # self-contained
registry_username=<RHN>
registry_password=<RHN_token>
postgresql_admin_password=<senha>
redis_mode=standalone                 # ou sentinel para HA

gateway_admin_password=<senha>
gateway_pg_host=<host_db>
gateway_pg_password=<senha>

controller_admin_password=<senha>
controller_pg_host=<host_db>
controller_pg_password=<senha>

hub_admin_password=<senha>
hub_pg_host=<host_db>
hub_pg_password=<senha>
hub_seed_collections=false            # true = 32GB + 45min

eda_admin_password=<senha>
eda_pg_host=<host_db>
eda_pg_password=<senha>
```

## Portas Essenciais no Firewall

```
443/tcp   → Gateway (acesso externo)
5432/tcp  → PostgreSQL (externo)
6379/tcp  → Redis
16379/tcp → Redis HA cluster bus
27199/tcp → Automation Mesh (Receptor)
```

## PostgreSQL

| Tipo | Versões | Obs |
|---|---|---|
| Gerenciado | 15 | Automático pelo installer |
| Externo | 15, 16, 17 | Requer ICU; 16/17 = backup manual |

## Operator OCP — Limites PostgreSQL Gerenciado

```
Não usar PostgreSQL gerenciado se:
  > 1 réplica de qualquer componente
  > 100 jobs concorrentes
  > 100 conexões ao banco
  > 100 GB storage em 30 dias
```
