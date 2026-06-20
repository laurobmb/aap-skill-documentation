# Ch04 — Exemplos Práticos de Uso da API

## Variável Base

```bash
export GATEWAY="https://aap.empresa.org"
export TOKEN="<seu-oauth2-token>"
export H="Authorization: Bearer $TOKEN"
export CT="Content-Type: application/json"
```

## Listar Job Templates

```bash
# Todos os JTs da organização Infraestrutura:
curl -k -H "$H" \
  "$GATEWAY/api/controller/v2/job_templates/?organization__name=Infraestrutura"

# Busca por nome:
curl -k -H "$H" \
  "$GATEWAY/api/controller/v2/job_templates/?name__icontains=deploy"

# Com paginação:
curl -k -H "$H" \
  "$GATEWAY/api/controller/v2/job_templates/?limit=50&offset=0"
```

## Lançar Job Template

```bash
# Lançar JT por ID:
curl -k -X POST -H "$H" -H "$CT" \
  -d '{}' \
  "$GATEWAY/api/controller/v2/job_templates/14/launch/"

# Com parâmetros extras:
curl -k -X POST -H "$H" -H "$CT" \
  -d '{
    "limit": "webservers",
    "extra_vars": {"app_version": "2.1.0", "env": "prod"},
    "tags": "deploy",
    "skip_tags": "cleanup"
  }' \
  "$GATEWAY/api/controller/v2/job_templates/14/launch/"

# Por Named URL (sem PK):
curl -k -X POST -H "$H" -H "$CT" \
  -d '{}' \
  "$GATEWAY/api/controller/v2/job_templates/Deploy+App++Infraestrutura/launch/"
```

## Monitorar Status do Job

```bash
# Ver detalhes do job lançado:
JOB_ID=<id-retornado-pelo-launch>
curl -k -H "$H" "$GATEWAY/api/controller/v2/jobs/$JOB_ID/"

# Filtrar jobs por status:
curl -k -H "$H" "$GATEWAY/api/controller/v2/jobs/?status=failed&order_by=-started"

# Jobs das últimas 24h:
SINCE=$(date -u -d '24 hours ago' '+%Y-%m-%dT%H:%M:%SZ')
curl -k -H "$H" "$GATEWAY/api/controller/v2/jobs/?started__gte=$SINCE"

# Output do job:
curl -k -H "$H" "$GATEWAY/api/controller/v2/jobs/$JOB_ID/stdout/?format=txt"
```

## Gerenciar Inventários

```bash
# Listar inventários:
curl -k -H "$H" "$GATEWAY/api/controller/v2/inventories/"

# Criar inventário:
curl -k -X POST -H "$H" -H "$CT" \
  -d '{
    "name": "Inventário Produção",
    "organization": 2,
    "description": "Servidores de produção"
  }' \
  "$GATEWAY/api/controller/v2/inventories/"

# Listar hosts de um inventário:
curl -k -H "$H" "$GATEWAY/api/controller/v2/inventories/5/hosts/"

# Filtrar hosts por fatos:
curl -k -H "$H" \
  "$GATEWAY/api/controller/v2/hosts/?host_filter=ansible_facts__ansible_distribution=RedHat"
```

## Lançar Workflow Job Template

```bash
curl -k -X POST -H "$H" -H "$CT" \
  -d '{
    "extra_vars": {"env": "producao", "version": "3.0"},
    "limit": "datacenter-sp"
  }' \
  "$GATEWAY/api/controller/v2/workflow_job_templates/8/launch/"
```

## Criar Projeto

```bash
curl -k -X POST -H "$H" -H "$CT" \
  -d '{
    "name": "Playbooks Infraestrutura",
    "organization": 2,
    "scm_type": "git",
    "scm_url": "https://github.com/empresa/ansible-infra.git",
    "scm_branch": "main",
    "scm_update_on_launch": true
  }' \
  "$GATEWAY/api/controller/v2/projects/"
```

## Script Python — Lançar JT e Aguardar

```python
import requests
import time

GATEWAY = "https://aap.empresa.org"
TOKEN = "seu-token-oauth2"
JT_ID = 14

headers = {
    "Authorization": f"Bearer {TOKEN}",
    "Content-Type": "application/json"
}

# Lançar job
resp = requests.post(
    f"{GATEWAY}/api/controller/v2/job_templates/{JT_ID}/launch/",
    json={"extra_vars": {"env": "prod"}},
    headers=headers,
    verify=False
)
job_id = resp.json()["id"]
print(f"Job lançado: {job_id}")

# Aguardar conclusão
while True:
    job = requests.get(
        f"{GATEWAY}/api/controller/v2/jobs/{job_id}/",
        headers=headers,
        verify=False
    ).json()
    status = job["status"]
    print(f"Status: {status}")
    if status in ("successful", "failed", "error", "canceled"):
        break
    time.sleep(10)

print(f"Job {job_id} terminou com status: {status}")
```
