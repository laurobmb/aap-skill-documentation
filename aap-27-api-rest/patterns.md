# Patterns — AAP 2.7 API REST

## Pattern 1: CI/CD Pipeline — Lançar JT e Aguardar Resultado

```bash
#!/bin/bash
# ci-deploy.sh — Usar em GitHub Actions, GitLab CI, Jenkins, etc.
set -euo pipefail

GATEWAY="${AAP_GATEWAY_URL}"
TOKEN="${AAP_DEPLOY_TOKEN}"
JT_NAME="Deploy App++Infraestrutura"
LIMIT="${TARGET_HOSTS:-}"
ENV="${DEPLOY_ENV:-staging}"

H="Authorization: Bearer $TOKEN"
CT="Content-Type: application/json"

# Lançar job
JOB_RESP=$(curl -sf -X POST \
  -H "$H" -H "$CT" \
  -d "{\"limit\": \"$LIMIT\", \"extra_vars\": {\"env\": \"$ENV\"}}" \
  "$GATEWAY/api/controller/v2/job_templates/$(python3 -c "import urllib.parse; print(urllib.parse.quote('$JT_NAME', safe=''))")/launch/")

JOB_ID=$(echo "$JOB_RESP" | jq -r '.id')
echo "Job $JOB_ID lançado"

# Aguardar e monitorar
while true; do
  STATUS=$(curl -sf -H "$H" "$GATEWAY/api/controller/v2/jobs/$JOB_ID/" | jq -r '.status')
  echo "Status: $STATUS"
  case "$STATUS" in
    successful) echo "✓ Deploy bem-sucedido"; exit 0 ;;
    failed|error|canceled) echo "✗ Deploy falhou: $STATUS"; exit 1 ;;
  esac
  sleep 15
done
```

## Pattern 2: Inventário Dinâmico via API para Portas de Entrada Específicas

```python
import requests

GATEWAY = "https://aap.empresa.org"
TOKEN = "token-automacao"

headers = {"Authorization": f"Bearer {TOKEN}", "Content-Type": "application/json"}

# Criar inventário
inv_resp = requests.post(
    f"{GATEWAY}/api/controller/v2/inventories/",
    json={"name": "Auto-Staging", "organization": 2},
    headers=headers, verify=False
)
inv_id = inv_resp.json()["id"]

# Popular com hosts
for host in ["app01.staging.org", "app02.staging.org", "db01.staging.org"]:
    requests.post(
        f"{GATEWAY}/api/controller/v2/hosts/",
        json={"name": host, "inventory": inv_id, "variables": '{"ansible_user": "ansible"}'},
        headers=headers, verify=False
    )

print(f"Inventário {inv_id} populado com {3} hosts")
```

## Pattern 3: Webhook Externo → API AAP (sem EDA)

```python
# Flask webhook que recebe evento e lança JT via API AAP
from flask import Flask, request
import requests, os

app = Flask(__name__)
GATEWAY = os.environ["AAP_GATEWAY"]
TOKEN = os.environ["AAP_TOKEN"]

@app.route("/webhook/deploy", methods=["POST"])
def deploy_webhook():
    payload = request.json
    env = payload.get("environment", "staging")
    version = payload.get("version", "latest")

    resp = requests.post(
        f"{GATEWAY}/api/controller/v2/job_templates/Deploy App++Infra/launch/",
        json={"extra_vars": {"env": env, "version": version}},
        headers={"Authorization": f"Bearer {TOKEN}"},
        verify=False
    )
    job_id = resp.json().get("id")
    return {"job_id": job_id, "status": "launched"}, 202
```
