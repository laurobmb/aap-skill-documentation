# Ch05 — Metrics Service API

## Endpoints e Base URL

```bash
# Containerizado (via Gateway):
https://<gateway>/api/metrics/v1/

# OCP Operator (via Gateway):
https://<gateway>/api/v1/

# Autenticação: Bearer token do Platform Gateway
curl -H "Authorization: Bearer <token>" \
  https://<gateway>/api/metrics/v1/health/

# Acesso local sem autenticação (troubleshooting):
# Containerizado:
podman exec automation-metrics-web curl http://localhost:8006/api/v1/health/

# OCP:
oc exec deployment/metrics-service-web -n <namespace> -- \
  curl http://localhost:8080/api/v1/health/
```

## /api/v1/health/ — Status de Saúde

```bash
# Sem autenticação (acesso direto ao container/pod)
curl http://localhost:8087/health/

# Via Gateway:
curl -H "Authorization: Bearer <token>" \
  https://<gateway>/api/metrics/v1/health/
```

**Resposta:**

```json
{
  "status": "good",
  "version": "1.0.0",
  "database": {
    "status": "connected",
    "migrations": "up-to-date"
  },
  "scheduler": {
    "status": "running",
    "last_heartbeat": "2026-05-20T14:35:00Z"
  }
}
```

**Status possíveis:**

| Status | Significado | Ação |
|---|---|---|
| `good` | Todos os componentes saudáveis | Nenhuma ação necessária |
| `degraded` | Alguns componentes com problema mas serviço funcional | Investigar componentes degradados |
| `unhealthy` | Componentes críticos falhando, serviço não funcional | Troubleshooting imediato |

**Códigos HTTP:**
- `200 OK` → serviço saudável
- `503 Service Unavailable` → degraded ou unhealthy
- `401 Unauthorized` → token inválido/ausente (acesso via Gateway)

**Integração com Prometheus/Nagios:**

```yaml
# Prometheus blackbox exporter ou http_probe:
http_probe:
  targets:
    - https://<gateway>/api/metrics/v1/health/
  bearer_token: <token>
```

## /api/v1/tasks/ — Status de Tarefas

```bash
# Listar tarefas (requer auth):
curl -H "Authorization: Bearer <token>" \
  https://<gateway>/api/metrics/v1/tasks/

# Filtros disponíveis:
?name=collect_hourly_metrics     # filtrar por nome da tarefa
?status=failure                  # filtrar por status (success/failure/running/pending)
?limit=100                       # máx resultados (padrão 50, máx 500)
?offset=50                       # paginação

# Ver tarefas com falha recente:
curl -H "Authorization: Bearer <token>" \
  "https://<gateway>/api/metrics/v1/tasks/?status=failure&limit=20"
```

**Resposta:**

```json
{
  "count": 150,
  "next": "/api/metrics/v1/tasks/?limit=50&offset=50",
  "results": [
    {
      "id": 1234,
      "name": "collect_hourly_metrics",
      "status": "success",
      "started": "2026-05-20T14:05:00Z",
      "finished": "2026-05-20T14:06:32Z",
      "duration_seconds": 92,
      "error_message": null
    }
  ]
}
```

## /api/v1/feature-flags/ — Flags de Funcionalidade

```bash
# Ver feature flags ativos:
curl -H "Authorization: Bearer <token>" \
  https://<gateway>/api/metrics/v1/feature-flags/
```

## Integração de Monitoramento

```bash
# Script de health check para monitoramento externo:
#!/bin/bash
GATEWAY="https://aap.empresa.org"
TOKEN="$AAP_MONITORING_TOKEN"

STATUS=$(curl -s -H "Authorization: Bearer $TOKEN" \
  "$GATEWAY/api/metrics/v1/health/" | jq -r '.status')

if [ "$STATUS" = "good" ]; then
  echo "OK - AAP Metrics Service saudável"
  exit 0
elif [ "$STATUS" = "degraded" ]; then
  echo "WARNING - AAP Metrics Service degradado"
  exit 1
else
  echo "CRITICAL - AAP Metrics Service unhealthy: $STATUS"
  exit 2
fi
```
