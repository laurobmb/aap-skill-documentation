# Ch03 — Instalação no OpenShift (Operator)

## Requisitos por Topologia Operator

### Operator Growth (SNO — Single Node OpenShift)

| Requisito | Valor |
|---|---|
| Cluster | Single Node OpenShift (SNO) |
| RAM | 32 GB |
| CPUs | 16 |
| Disco | 128 GB |
| IOPS | 3000 |
| OpenShift | 4.14+ |
| ansible-core | 2.16+ |

### Operator Enterprise (Cluster Multi-node)

- 3 primary nodes + 2 worker nodes
- PostgreSQL externo (4 vCPUs, 16 GB RAM mínimo) — **recomendado**
- Redis externo — **recomendado**

## Pods Criados (Operator Growth)

| Pod | Função |
|---|---|
| automation controller web | UI e API do Controller |
| automation controller task | Execução de jobs |
| automation hub web + API | Interface e API do Hub |
| automation hub content (×2) | Servir coleções e EEs |
| automation hub worker (×2) | Tarefas assíncronas do Hub |
| automation hub Redis | Cache do Hub |
| EDA API | API do Event-Driven Ansible |
| EDA activation worker | Worker de ativações EDA |
| EDA default worker | Worker padrão EDA |
| EDA event stream | Stream de eventos EDA |
| EDA scheduler | Agendamento EDA |
| Platform gateway | Proxy e autenticação central |
| Database | PostgreSQL gerenciado |
| Redis | Cache geral |

## Custom Resource File — Exemplo Operator Growth

```yaml
apiVersion: aap.ansible.com/v1alpha1
kind: AnsibleAutomationPlatform
metadata:
  name: myaap
  namespace: aap
spec:
  # Banco de dados gerenciado (somente para growth/desenvolvimento)
  # Para produção, usar PostgreSQL externo
  database:
    postgres_storage_class: standard
    postgres_storage_requirements:
      requests:
        storage: 100Gi

  # Lightspeed (opcional)
  lightspeed:
    disabled: true

  # Automation controller
  controller:
    admin_password_secret: controller-admin-secret

  # Hub
  hub:
    admin_password_secret: hub-admin-secret

  # EDA
  eda:
    admin_password_secret: eda-admin-secret
```

## Custom Resource — PostgreSQL Externo (Produção)

```yaml
spec:
  database:
    database_secret: external-db-secret   # Secret com host, user, password, database
```

```yaml
# Secret de banco externo
apiVersion: v1
kind: Secret
metadata:
  name: external-db-secret
  namespace: aap
stringData:
  host: db.empresa.org
  port: "5432"
  database: aap
  username: aapuser
  password: <senha>
  sslmode: require
```

## Limitações do PostgreSQL Gerenciado pelo Operator

> **ATENÇÃO:** Não usar o PostgreSQL gerenciado pelo Operator se qualquer condição abaixo for verdadeira:
>
> - Mais de 1 réplica de qualquer componente (gateway, controller, hub, EDA)
> - Capacidade do controller > 100 jobs concorrentes
> - Total de conexões ao banco > 100
> - Storage estimado em 30 dias > 100 GB

Para esses casos, usar **PostgreSQL externo**.

## Múltiplos Deployments no Mesmo Cluster OCP

```yaml
# Namespace-scoped deployment — permite múltiplas instâncias AAP no mesmo cluster
# Cada namespace = 1 instância isolada
# Apenas 1 Operator por namespace (não instalar 2 no mesmo namespace)

# Criar namespace separado para cada instância
oc new-project aap-producao
oc new-project aap-homologacao
```

## Instalação via OperatorHub (CLI)

```bash
# 1. Instalar o Operator AAP via CLI
oc apply -f - <<EOF
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: ansible-automation-platform-operator
  namespace: aap
spec:
  channel: stable-2.7
  name: ansible-automation-platform-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF

# 2. Criar custom resource após Operator estar running
oc apply -f aap-cr.yaml -n aap

# 3. Acompanhar status
oc get ansibleautomationplatform -n aap -w
```
