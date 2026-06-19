# Ch02 — Performance Tuning no OpenShift (OCP)

## Resource Requests e Limits

```yaml
# Custom pod specification para Container Group
# Controller → Infrastructure → Instance Groups → Pod Spec Override
apiVersion: v1
kind: Pod
spec:
  containers:
    - name: worker
      resources:
        requests:
          memory: "512Mi"
          cpu: "250m"
        limits:
          memory: "2Gi"
          cpu: "2000m"
```

## Ajustar Control Plane (Task Container)

```yaml
# AutomationController custom resource
spec:
  task_resource_requirements:
    requests:
      cpu: "2000m"
      memory: "4Gi"
    limits:
      cpu: "4000m"
      memory: "8Gi"
  web_resource_requirements:
    requests:
      cpu: "500m"
      memory: "1Gi"
    limits:
      cpu: "2000m"
      memory: "4Gi"
```

## Node Selectors — Dedicar Nós para Jobs

```yaml
# Atribuir jobs de execução a nós específicos (ex: nós com mais recursos)
spec:
  task_tolerations: |
    - key: "dedicated"
      operator: "Equal"
      value: "automation"
      effect: "NoSchedule"
  task_node_selector: |
    dedicated: automation
```

## Minimizar Downtime durante Upgrade do OCP

```yaml
# AutomationController CR
spec:
  ee_extra_env: |
    - name: RECEPTOR_KUBE_SUPPORT_RECONNECT
      value: enabled
  termination_grace_period_seconds: 3600   # esperar jobs terminarem
  task_affinity:
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - podAffinityTerm:
            labelSelector:
              matchExpressions:
                - key: app.kubernetes.io/name
                  operator: In
                  values: [awx-task]
            topologyKey: topology.kubernetes.io/zone
          weight: 100
```

```yaml
# PodDisruptionBudget para proteger jobs durante upgrade
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: automationcontroller-job-pods
spec:
  maxUnavailable: 0
  selector:
    matchExpressions:
      - key: ansible-awx-job-id
        operator: Exists
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: automationcontroller-web-pods
spec:
  minAvailable: 1
  selector:
    matchExpressions:
      - key: app.kubernetes.io/name
        operator: In
        values: [myaap-web]
```

## Timeout de Pod Pendente

```yaml
# Aumentar se pods demoram para iniciar por ResourceQuota
spec:
  extra_settings:
    - setting: AWX_CONTAINER_GROUP_POD_PENDING_TIMEOUT
      value: "7200"   # 2 horas (padrão)
```

## Réplicas por Componente (OCP)

```yaml
spec:
  automation_gateway_replicas: 2     # HA para gateway
  automation_controller_replicas: 2  # HA para controller
  automation_hub_api_replicas: 2
  automation_hub_content_replicas: 2
  automation_eda_api_replicas: 2
```

> Ao usar mais de 1 réplica de qualquer componente: usar **PostgreSQL externo** (banco gerenciado não suporta mais de 100 conexões).
