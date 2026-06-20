# Cheatsheet — AAP 2.7 API REST

## Endpoints Base

```
https://<gateway>/api/controller/v2/    Controller API
https://<gateway>/api/gateway/v1/       Platform Gateway API
https://<gateway>/api/eda/v1/           EDA API
https://<gateway>/api/galaxy/           Hub API
https://<gateway>/api/metrics/v1/       Metrics Service API
```

## Auth OAuth2 (Token)

```bash
# Criar token:
curl -u user:pass -X POST https://<gw>/api/gateway/v1/tokens/ -k

# Usar token:
curl -H "Authorization: Bearer <token>" https://<gw>/api/controller/v2/<endpoint>/
```

## Filtros Rápidos

```bash
?name__icontains=deploy        # contém (case-insensitive)
?status__in=failed,error       # múltiplos valores
?created__gte=2024-01-01       # data mínima
?not__status=successful        # NOT
?or__status=failed&or__status=error  # OR
?field__isnull=true            # campo nulo
```

## Operações Comuns

```bash
# Lançar JT por ID:
POST /api/controller/v2/job_templates/14/launch/

# Lançar JT por nome:
POST /api/controller/v2/job_templates/Nome++Org/launch/

# Status de job:
GET /api/controller/v2/jobs/<id>/

# Output de job:
GET /api/controller/v2/jobs/<id>/stdout/?format=txt
```

## Named URL Format

```
job_templates: <name>++<organization.name>
hosts:         <name>++<inventory.name>++<organization.name>
inventories:   <name>++<organization.name>
users:         <username>
```

## Metrics Health

```bash
GET /api/metrics/v1/health/   → status: good|degraded|unhealthy
GET /api/metrics/v1/tasks/?status=failure  → tarefas com falha
```
