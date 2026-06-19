# Cheatsheet — AAP 2.7 Operação

## Capacidade de Jobs

```
forks_mem = (RAM_MB - 2048) / 100
forks_cpu = CPUs * 4
capacidade = min(forks_mem, forks_cpu)
impacto_job = forks_job + 1

Exemplo: 16GB RAM, 8 CPUs
  mem: (16384-2048)/100 = 143
  cpu: 8*4 = 32  ← limitante
  → 32 forks disponíveis
```

## Coleta de Diagnóstico

```bash
# OCP
oc adm must-gather \
  --image=registry.redhat.io/ansible-automation-platform-26/aap-must-gather-rhel9 \
  --dest-dir /tmp/diag

# Containerizado
ansible-playbook -i inventory \
  ansible.containerized_installer.log_gathering \
  -e 'case_number=123456' -e 'upload=true'
```

## Logs por Container

```bash
podman logs automationcontroller-task   # jobs/tasks
podman logs automationcontroller-web    # API
podman logs automationgateway           # proxy/auth
podman logs automation-eda-api          # EDA
```

## HTTP 401 Pós-Upgrade → Criar PAT no Gateway

```bash
curl -X POST https://aap.exemplo.org/api/gateway/v1/tokens/ \
  -H "Authorization: Bearer <admin_token>" \
  -d '{"description": "meu token", "scope": "write"}'
```

## KPIs Críticos

```
CPU > 80%       → horizontal scale
API > 500ms     → adicionar gateway/controller nodes
Jobs pending    → verificar capacity + execution nodes
EDA failures    → verificar DE + escalar eda nodes
```

## metrics-utility Rápido

```bash
metrics-utility gather_automation_controller_data
metrics-utility build_report --since 2025-01-01 --until 2025-02-01
```
