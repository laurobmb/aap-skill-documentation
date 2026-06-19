# Ch03 — Observabilidade: Metrics, Analytics e Dashboard

## metrics-utility — Relatórios de Consumo/Billing

Ferramenta para gerar relatórios de uso do AAP para billing baseado em consumo.

### Instalar e Configurar (Containerizado)

```ini
# inventory file
[all:vars]
# Habilitar metrics-utility
metrics_utility_enabled=true
metrics_utility_install=true
metrics_utility_ship_target=directory   # directory | s3
metrics_utility_ship_path=/var/lib/awx/metrics-reports

# Para S3:
# metrics_utility_ship_target=s3
# metrics_utility_s3_bucket=meu-bucket-aws
# metrics_utility_s3_path=reports/aap/
```

### Executar relatório mensal

```bash
# Coletar dados do mês atual
metrics-utility gather_automation_controller_data

# Gerar relatório
metrics-utility build_report --since 2025-01-01 --until 2025-02-01

# Para relatório CCSP (Certified Cloud and Service Provider)
metrics-utility build_report --format ccsrv2 \
  --since 2025-01-01 --until 2025-02-01

# Filtrar por organização
metrics-utility build_report --organization "Financeiro" \
  --since 2025-01-01 --until 2025-02-01
```

### Configurar no OpenShift (Operator)

```yaml
# AnsibleAutomationPlatform CR — configurar metrics-utility via ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: metrics-utility-config
  namespace: aap
data:
  METRICS_UTILITY_SHIP_TARGET: "directory"
  METRICS_UTILITY_SHIP_PATH: "/var/data/metrics"
```

## Automation Analytics

Métricas de performance e ROI disponíveis em console.redhat.com/ansible/automation-analytics:

| Relatório | O que mostra |
|---|---|
| **Job Explorer** | Todos os jobs com filtros (org, template, status, host) |
| **Automation Calculator** | ROI: custo manual vs automatizado |
| **Savings Planner** | Projeção de savings futuros |
| **Host Explorer** | Hosts únicos gerenciados |
| **Top Templates** | Templates mais executados e seu ROI |

### Habilitar envio de dados

```ini
# inventory file (containerizado)
controller_analytics_enabled=true

# Ou via UI: Settings → Miscellaneous System → Insights for Ansible Automation Platform
```

## Automation Dashboard (Local)

Aplicação containerizada para métricas de **até 3 instâncias AAP** sem depender de conectividade externa.

```bash
# Instalação (RHEL + Podman)
cd /opt/automation-dashboard-installer
ansible-playbook -i inventory install.yml

# Configurar instances (clusters.yaml)
clusters:
  - name: "AAP Produção"
    url: "https://aap-prod.empresa.org"
    token: "<gateway_token>"
  - name: "AAP Homologação"
    url: "https://aap-hom.empresa.org"
    token: "<gateway_token>"

# Sincronizar dados
podman exec automation-dashboard ./manage.py setclusters
podman exec automation-dashboard ./manage.py syncdata
```

## Monitoring com Prometheus (OCP)

```yaml
# OCP expõe métricas do AAP em /metrics
# Configurar ServiceMonitor para coletar
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: aap-controller
  namespace: aap
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: myaap
  endpoints:
    - port: http
      path: /api/controller/v2/metrics/
      bearerTokenSecret:
        name: metrics-token
        key: token
```
